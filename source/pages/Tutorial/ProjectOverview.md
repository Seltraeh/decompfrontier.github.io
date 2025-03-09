# GimuFrontier Functionality Document
# Last Updated: March 09, 2025
# Notes: Covers all provided files. Format: [File] Function/Class: Description {Details}

---

## Main Components

[main.cpp] Main Entry Point: Initializes server with configs, sets up SQLite3, runs migrations, catches exceptions.
{ gimuconfig.json, config.json, System::Instance().GetDbPath(), drogon::app().getDbClient("gme") }

[System.hpp/.cpp] System (Singleton): Manages configs, migrations, paths.
- LoadSystemConfig(const std::string& p): Loads gimuconfig.json, parses system/server/log sections, creates log dirs.
{ m_contentRoot, m_dbPath, m_sessionTimeout, m_serverCfg, m_mstConfig, m_logCfg }
- RunMigrations(drogon::orm::DbClientPtr ptr): Executes registered migrations.
- Getters: GetContentRootPath(), GetDbPath(), GetSessionTimeout(), MstConfig(), ServerConfig(), LogConfig().
{ Singleton instance: System::m_sys }

[GmeController.hpp/.cpp] GmeController: Routes GME requests to handlers.
- HandleGame(HttpRequestPtr, Callback): Decrypts body, finds user in m_users, dispatches to handler via HEADER_REQUEST_ID.
{ Endpoints: /bf/gme/action.php, /bf/gme/featureCheck.php, /bf/gme/action/getServerTime.php, /bf/gme/action/Daily_login.php }
- InitializeHandlers(): Registers handlers with REGISTER macro.
{ Handlers: InitializeHandler, UserInfoHandler, etc.; m_users stores UserInfo by ID }
- m_users: Hardcoded user "0101AABB", placeholder for session management.

---

## Database and Migrations

[IMigration.hpp] IMigration: Abstract base for migrations.
- execute(DbClientPtr): Pure virtual, runs SQL.
- getName(): Pure virtual, returns migration ID.

[MigrationManager.hpp/.cpp] MigrationManager: Manages and runs migrations.
- Constructor: Calls Register().
- RunMigrations(DbClientPtr): Creates migration_status table, runs new migrations, logs execution.
- Register(): Adds CreateDefaultTables, CreateUserUnitsTable to m_migs.
- CreateGetMigrationStatus(DbClientPtr, vector<string>&): Initializes migration_status, fetches applied hashes.
{ m_migs: vector of IMigration shared_ptrs }

[CreateDefaultTables.hpp] CreateDefaultTables: Sets up users and userinfo tables.
- execute(DbClientPtr): Creates users (id PK, account_id, username, admin), userinfo (id PK, level, exp, etc.).
- getName(): "06022024_CreateUserInfoTable".
{ userinfo fields: level(3), exp(3), max_unit_count(9), zel(9), etc. }

[CreateUserUnitsTable.hpp] CreateUserUnitsTable: Sets up user_units table.
- execute(DbClientPtr): Creates user_units (user_id, unit_id PK, name, rarity, element, stats, bb, leader_skill, ai, data; all TEXT except rarity INTEGER).
- getName(): "08032025_CreateUserUnitsTable".

[DbMacro.hpp] GME_DB: Macro for drogon::app().getDbClient("gme").

---

## Requests and Responses

[GmeRequest.hpp] IRequest: Abstract base for request deserialization.
- getGroupName(): Pure virtual, returns group ID (e.g., "IKqx1Cn9").
- DeserializeFields(Json::Value, size_t): Pure virtual, parses JSON fields.

[UserInfo.hpp] UserInfo: Parses user info requests.
- getGroupName(): "IKqx1Cn9".
- DeserializeFields(): Maps obfuscated keys (e.g., "h7eY3sAK" -> userID) to fields.
{ Fields: mInfo, userID, contactID, gumiInfo(GumiLiveInfo), pointerName, modelChangeCount, deviceName, targetOS, etc. }

[GumiLiveInfo.hpp] GumiLiveInfo: Sub-struct for Gumi Live data.
- Deserialize(Json::Value): Maps keys (e.g., "iN7buP2h" -> gumiLiveUserID).
{ Fields: gumiLiveUserID, gumiLiveToken, facebookID, userID }

[MstUrlList.hpp] MstUrlList: Parses master URL list requests.
- getGroupName(): "KeC10fuL".
- DeserializeFields(): Maps "moWQ30GH" -> id, "d2RFtP8T" -> version.

[GmeTypes.hpp] UserInfo: Aggregates Response::UserInfo and Response::UserTeamInfo.
{ Hardcoded userID "0101AABB" }

---

## Handlers

[GmeHandler.hpp] HandlerBase: Abstract base for request handlers.
- GetGroupId(): Pure virtual, returns group ID (e.g., "cTZ3W2JG").
- GetAesKey(): Pure virtual, returns AES key (e.g., "ScJx6ywWEb0A3njT").
- Handle(UserInfo&, DrogonCallback, Json::Value): Pure virtual, processes request.

[UserInfoHandler.hpp/.cpp] UserInfoHandler: Handles user info requests.
- GetGroupId(): "cTZ3W2JG".
- GetAesKey(): "ScJx6ywWEb0A3njT".
- Handle(): Overrides userID (from req["user_id"], req["ak"], or "0839899613932562"), handles actions (load_unit_inventory, Zw3WIoWu, load_squad_management) or initial load.
{ Queries: user_units (LIMIT 500), userinfo; Builds: UserUnitInfo, UserPartyDeckInfo, etc.; Dummy unit if empty }

[DeckEditHandler.hpp/.cpp] DeckEditHandler: Handles deck edits (stub).
- GetGroupId(): "m2Ve9PkJ".
- GetAesKey(): "d7UuQsq8".
- Handle(): Returns user info, SignalKey ("fZnLr4t9"), NoticeInfo with URL.
{ No req processing yet }

[BadgeInfoHandler.hpp/.cpp] BadgeInfoHandler: Handles badge info requests.
- GetGroupId(): "nJ3A7qFp".
- GetAesKey(): "bGxX67KB".
- Handle(): Serializes Response::BadgeInfo, returns via newGmeOkResponse.
{ Minimal implementation, likely placeholder }

[ControlCenterEnterHandler.hpp/.cpp] ControlCenterEnterHandler: Handles control center entry (slot game).
- GetGroupId(): "uYF93Mhc".
- GetAesKey(): "d0k6LGUu".
- Handle(): Serializes UserTeamInfo and Response::SlotGameInfoR (hardcoded slot data: Brave Slots, reelPos, images, etc.).
{ SlotGameInfoR: id=1, name="Brave Slots", reelPos="1,2,3", useMedal=1, slotHelpUrl="/bf/web/slots/html/index.php", multiple picture entries }

[FriendGetHandler.hpp/.cpp] FriendGetHandler: Handles friend/reinforcement info requests.
- GetGroupId(): "2o4axPIC".
- GetAesKey(): "EoYuZ2nbImhCU1c0".
- Handle(): Serializes Response::ReinforcementInfo, returns via newGmeOkResponse.
{ Minimal implementation, likely placeholder }

[GatchaListHandler.hpp/.cpp] GatchaListHandler: Handles gacha list requests.
- GetGroupId(): "Uo86DcRh".
- GetAesKey(): "8JbxFvuSaB2CK7Ln".
- Handle(): Adds SignalKey ("axhp8Sin"), UserTeamInfo, and MstConfig gacha info to response.
{ Uses System::Instance().MstConfig().CopyGachaInfoTo() }

[HomeInfoHandler.hpp/.cpp] HomeInfoHandler: Handles home screen info (stub).
- GetGroupId(): "NiYWKdzs".
- GetAesKey(): "f6uOewOD".
- Handle(): Empty implementation.
{ Placeholder, likely incomplete }

[MissionStartHandler.hpp/.cpp] MissionStartHandler: Handles mission start requests (stub).
- GetGroupId(): "jE6Sp0q4".
- GetAesKey(): "csiVLDKkxEwBfR70".
- Handle(): Returns empty JSON response via newGmeOkResponse.
{ Placeholder, no logic implemented }

[InitializeHandler.hpp/.cpp] InitializeHandler: Handles game initialization.
- GetGroupId(): "MfZyu1q9".
- GetAesKey(): "EmcshnQoDr20TZz1".
- Handle(): Deserializes Request::UserInfo, queries users table, calls OnUserInfoSuccess.
- OnUserInfoSuccess(Result, Callback, UserInfo&): Populates UserInfo from DB or defaults (handleName="BFOM: 03/03/2024", random accountID), serializes extensive response.
{ Response includes: UserInfo, SignalKey("C7vnXA5T"), ChallengeArenaUserInfo, DailyTaskBonusMst, DailyTaskPrizeMst, GuildInfo, DailyTaskMst (hardcoded tasks: AV, VV, CM), DailyLoginRewardsUserInfo, VideoAdsSlotgameInfo, SummonerJournalUserInfo, MstConfig data }

[UpdateInfoLightHandler.hpp/.cpp] UpdateInfoLightHandler: Handles light info updates (stub).
- GetGroupId(): "ynB7X5P9".
- GetAesKey(): "7kH9NXwC".
- Handle(): Returns empty JSON response via newGmeOkResponse.
{ Placeholder, no logic implemented }

---

## Responses

[ArenaRankMst.hpp] ArenaRankMst: Response for arena rank master data.
- getGroupName(): "6kWq78zx".
- Data: id, rank_point_start, rank_point_end, reward_type, name, reward_param, scenario_info; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=0, rank_point_end=0, rank_point_start=0, reward_type=0, strings empty }

[BadgeInfo.hpp] BadgeInfo: Response for badge info.
- getGroupName(): "h23iRjGN".
- isArray(): False (implied).
- Fields: scenarioNum, unitDictNum, itemDictNum, newFrohun, dungeonKeyNum, badgeData; serializes to JSON with obfuscated keys.
- SerializeFields(): Outputs fields to JSON.
{ Non-array response, default constructor: all numeric fields 0, badgeData empty; replaces earlier assumption }

[DailyLoginRewardsUserInfo.hpp] DailyLoginRewardsUserInfo: Response for daily login rewards user info.
- getGroupName(): "Drudr2w5".
- isArray(): False.
- Fields: id, userCurrentCount, userLimitCount, currentDay, guaranteedRemainigDays, message, nextRewardId; serializes to JSON with obfuscated keys.
- SerializeFields(): Outputs fields to JSON, nextRewardId conditional.
{ Non-array response, default constructor: id=1, userCurrentCount=0, userLimitCount=0, currentDay=1, guaranteedRemainigDays=0, message=" day(s) more to guaranteed Gem!", nextRewardId=0; replaces earlier assumption }

[ChallengeArenaUserInfo.hpp] ChallengeArenaUserInfo: Response for challenge arena user info.
- getGroupName(): "XGmGpmYW".
- isArray(): False (implied).
- Fields: unkint, unkint2, rainbowCoins, unkint4, unkint5, leagueId, unkint7, unkint8, unkint9, unkstr, unkstr2; serializes to JSON with obfuscated keys.
- SerializeFields(): Outputs fields to JSON, TODO for full decomp.
{ Non-array response, default constructor: all numeric fields 0, strings empty; replaces earlier assumption }

[BannerInfoMst.hpp] BannerInfoMst: Response for banner info master data.
- getGroupName(): "Pk5F1vhx".
- Data: id, readCount, dispOrder, name, url, image, param, pageType, targetOS; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=1, readCount=0, dispOrder=0, strings empty; replaces earlier assumption }

[DailyTaskBonusMst.hpp] DailyTaskBonusMst: Response for daily task bonus master data.
- getGroupName(): "p283g07d".
- Data: bonusBravePoints; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: bonusBravePoints=0; replaces earlier assumption }

[ExcludedDungeonMissionMst.hpp] ExcludedDungeonMissionMst: Response for excluded dungeon mission master data.
- getGroupName(): "3aDk1xk7".
- Data: id; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=0 }

[DailyTaskMst.hpp] DailyTaskMst: Response for daily task master data.
- getGroupName(): "k23D7d43".
- Data: typeKey, typeTitle, typeDescription, taskCount, taskBravePoints, bravePointsTotal, bravePoints, taskAreaIDs, timesCompleted; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: numeric fields 0, strings empty; replaces earlier assumption }

[DungeonKeyMst.hpp] DungeonKeyMst: Response for dungeon key master data.
- getGroupName(): "4NG79sX1".
- Data: id, dungeonId, limitSec, possessionLimit, distributeCount, distributeFlag, state, name, thumImg, openImg, closeImg, usagePattern; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=1, others 0 or empty; distributeCount optional }

[EventTokenInfo.hpp] EventTokenInfo: Response for event token info.
- getGroupName(): "l234vdKs".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[FeatureGatingInfo.hpp] FeatureGatingInfo: Response for feature gating info.
- getGroupName(): "2375D38i".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[FirstDescMst.hpp] FirstDescMst: Response for first description master data.
- getGroupName(): "8UawtzHw".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[DailyTaskPrizeMst.hpp] DailyTaskPrizeMst: Response for daily task prize master data.
- getGroupName(): "a739yK18".
- Data: taskId, presentType, targetId, targetCount, bravePointCost, maxClaimCount, currentClaimCount, timeLimit, isMileStonePrize, taskTitle, taskDesc, targetParam; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: numeric fields 0, timeLimit=0, isMileStonePrize=false, strings empty; replaces earlier assumption }

[FeatureCheck.hpp] FeatureCheck: Response for feature check flags.
- getGroupName(): nullptr.
- isArray(): False.
- Fields: Numerous boolean flags (randall, frontierhunter, etc.), battle_item_limit, dungeon_key_cnt, cooldown_timer, daily_dungeon_list; serializes to JSON with direct keys.
- SerializeFields(): Outputs fields to JSON, includes hardcoded extras (e.g., dungeon_key=1).
{ Non-array response, default constructor sets extensive defaults (e.g., randall=true, battle_item_limit=500) }

[DefineMst.hpp] DefineMst: Response for define master data.
- getGroupName(): "VkoZ5t3K".
- isArray(): False.
- Fields: Extensive list of uint64_t, float, uint32_t, and string limits/rates (maxZel, friendPointReinFriendRate, etc.); serializes to JSON with obfuscated keys.
- SerializeFields(): Outputs fields to JSON, some optional (e.g., actionPointRecoverFixed).
{ Non-array response, default constructor: all fields 0 or empty }

[ExtraPassiveSkillMst.hpp] ExtraPassiveSkillMst: Response for extra passive skill master data.
- getGroupName(): "?".
- Data: id, target, priority, rare, groupID, skillType, skillName, skillNameS, processParam, termParam, description, processID; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: numeric fields 0, strings empty }

[GachaEffectMst.hpp] GachaEffectMst: Response for gacha effect master data.
- getGroupName(): "Pf97SzVw".
- Data: id, gatcha_id, rare, rate, effect_before, effect_after, effect; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=0, gatcha_id=0, rare=0, rate=0.0f, strings empty }

[GachaCategory.hpp] GachaCategory: Response for gacha category data.
- getGroupName(): "IBs49NiH".
- Data: id, dispOrder, startDate, endDate, img, gachaIDList; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=0, dispOrder=0, startDate=0, endDate=0, strings empty }

[GachaInfo.hpp] GachaInfo: Response for gacha info.
- getGroupName(): "1IR86sAv".
- Data: id, braveCoin, resummonGachaFlag, gachaType, priority, needFriendPoint, onceDayFlag, gachaDetailID, gachaGroupID, btnImg, bgImg, endDate, captionMsg, detailMsg, gachaName, campaignInfo, startDate, startHour, endHour, doorImg, description, commentMsg, contentsBannerImg; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: all numeric fields 0, strings empty; gachaDetailID optional empty string }

[GiftItemMst.hpp] GiftItemMst: Response for gift item master data.
- getGroupName(): "Bm1WNDQ0".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[FriendPointInfo.hpp] FriendPointInfo: Response for friend point info.
- getGroupName(): "6e4b7sQt".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[GeneralEventMst.hpp] GeneralEventMst: Response for general event master data.
- getGroupName(): "Md9gG3c0".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[GachaMst.hpp] GachaMst: Response for gacha master data.
- getGroupName(): "5Y4GJeo3".
- Data: id, type, priority, need_friend_point, need_gems, once_day_flag, gatcha_detail_id, gatcha_group_id, name, start_date, end_date, start_hour, end_hour, bg_img, btn_img, door_img, caption_msg, detail_msg, comment_msg, description, contents_banner_img; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: all numeric fields 0, strings empty; gatcha_detail_id and contents_banner_img optional }

[ItemFavorite.hpp] ItemFavorite: Response for item favorites.
- getGroupName(): "VSRPkdId".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[LoginCampaignMst.hpp] LoginCampaignMst: Response for login campaign master data.
- getGroupName(): "Bd29Pqw0".
- isArray(): False (implied).
- Fields: id, startDate, totalDays, image; serializes to JSON with obfuscated keys.
- SerializeFields(): Outputs fields to JSON.
{ Non-array response, default constructor: id=1, startDate=0, totalDays=0, image empty }

[GuildInfo.hpp] GuildInfo: Response for guild info.
- getGroupName(): "IkdSufj5".
- isArray(): False.
- Fields: id; serializes to JSON with obfuscated keys, TODO for additional fields if id != 0.
- SerializeFields(): Outputs id to JSON.
{ Non-array response, default constructor: id=0; replaces earlier assumption }

[NpcMessageOverwriteInfo.hpp] NpcMessageOverwriteInfo: Response for NPC message overwrite info.
- getGroupName(): "yNnvj59x".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[LoginCampaignReward.hpp] LoginCampaignReward: Response for login campaign rewards.
- getGroupName(): "bD18x9Ti".
- Data: id, reward_day, reward_img; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=1, reward_day=0, reward_img empty }

[NpcTeamInfo.hpp] NpcTeamInfo: Response for NPC team info.
- getGroupName(): nullptr.
- isArray(): False (implied).
- Fields: userId, friendMessage, lv; serializes to JSON with prefixed keys.
- SerializeFields(): Outputs fields to JSON.
{ Non-array response, default constructor: lv=0, strings empty }

[NpcUnitInfo.hpp] NpcUnitInfo: Response for NPC unit info.
- getGroupName(): nullptr.
- Data: id, party_id, type, lv, hp, atk, def, hel, skill_id, skill_lv, equip_item_id; serializes to JSON with prefixed keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: all fields 0; skill_id optional empty string }

[MissionBreakInfo.hpp] MissionBreakInfo: Response for mission break info.
- getGroupName(): "5PR2VmH1".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[PermitPlace.hpp] PermitPlace: Response for permit place info.
- getGroupName(): "yXNM8kL3".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[NpcPartyInfo.hpp] NpcPartyInfo: Response for NPC party info.
- getGroupName(): nullptr.
- Data: id, type, disp_order; serializes to JSON with prefixed keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: all fields 0 }

[Unknown_Empty.hpp] Unknown: Empty file, assumed placeholder.
- getGroupName(): N/A.
- Data: N/A.
- Mst: N/A.
{ Placeholder, no content provided }

[NoticeInfo.hpp] NoticeInfo: Response for notice info.
- getGroupName(): "Pj6zDW3m".
- isArray(): False.
- Fields: id, url; serializes to JSON with obfuscated keys.
- SerializeFields(): Outputs fields to JSON.
{ Non-array response, default constructor: id=1, url empty; replaces earlier assumption }

[NpcMst.hpp] NpcMst: Response for NPC master data.
- getGroupName(): "hV5vWu6C".
- Data: id, arena_rank_id, handle_name, team(NpcTeamInfo), units(NpcUnitInfo), parties(NpcPartyInfo); serializes to JSON with obfuscated keys and nested serialization.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=0, arena_rank_id=0, handle_name empty; dependencies: NpcTeamInfo.hpp, NpcUnitInfo.hpp, NpcPartyInfo.hpp }

[PermitPlaceML.hpp] PermitPlaceML: Response for permit place ML info.
- getGroupName(): "Y73mHKS8".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[PermitPlaceSp.hpp] PermitPlaceSp: Response for permit place Sp info.
- getGroupName(): "Y73tHKS8".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[RaidUserInfo.hpp] RaidUserInfo: Response for raid user info.
- getGroupName(): "1ry6BKoe".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[ReceipeMst.hpp] ReceipeMst: Response for recipe master data.
- getGroupName(): "8f0bCciN".
- Data: id, itemId, itemCount, karma, unk, materials, keyItemIds, unk2, unk, unk3, unk4; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=0, itemId=0, itemCount=0, karma=0, all strings empty; TODO: needs full decomp for unknowns }

[ReinforcementInfo.hpp] ReinforcementInfo: Response for friend/reinforcement info.
- getGroupName(): "xZH6EIQ7".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, replaces earlier assumption from FriendGetHandler }

[ResummonGatchaMst.hpp] ResummonGatchaMst: Response for resummon gacha master data.
- getGroupName(): "da3qD39b".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[SignalKey.hpp] SignalKey: Response for signal key.
- getGroupName(): "6FrKacq7".
- Fields: key (string).
- SerializeFields(): Outputs key to JSON with obfuscated key "Kn51uR4Y".
{ Non-array response, replaces earlier assumption; used in multiple handlers (e.g., "fZnLr4t9", "axhp8Sin", "C7vnXA5T") }

[SlotGameInfo.hpp] SlotGameInfo: Response for slot game info.
- getGroupName(): nullptr.
- isArray(): False.
- Fields: id, name, reelPos, useMedal, slotHelpUrl, slotImage; serializes to JSON with obfuscated keys.
- SerializeFields(): Outputs fields to JSON.
{ Non-array response, default constructor not explicit, fields uninitialized }

[SlotGameInfoR.hpp] SlotGameInfoR: Response for slot game info (ControlCenterEnterHandler variant).
- getGroupName(): "C38FmiUn".
- Fields: info(SlotGameInfo), pictures(SlotGamePictureInfo).
- SerializeFields(): Serializes info and pictures to JSON with obfuscated keys, uses StreamWriterBuilder for formatting.
{ Non-array response, dependencies: SlotGameInfo.hpp, SlotGamePictureInfo.hpp; matches earlier submission }

[SummonerJournalUserInfo.hpp] SummonerJournalUserInfo: Response for summoner journal user info.
- getGroupName(): "M3dw18eB".
- isArray(): False.
- Fields: userId, points, journalFlag.
- SerializeFields(): Outputs fields to JSON with obfuscated keys.
{ Non-array response, default constructor: points=0, journalFlag=0, userId empty }

[SlotGameInfoR.hpp] SlotGameInfoR: Response for slot game info (ControlCenterEnterHandler variant).
- getGroupName(): "C38FmiUn".
- Fields: info(SlotGameInfo), pictures(SlotGamePictureInfo).
- SerializeFields(): Serializes info and pictures to JSON with obfuscated keys, uses StreamWriterBuilder for formatting.
{ Non-array response, dependencies: SlotGameInfo.hpp, SlotGamePictureInfo.hpp }

[SummonTicketV2Mst.hpp] SummonTicketV2Mst: Response for summon ticket V2 master data.
- getGroupName(): "hE1d083b".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[SlotGameReelInfo.hpp] SlotGameReelInfo: Response for slot game reel info.
- getGroupName(): nullptr.
- Data: id, reelData; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=0, reelData empty }

[TownLocationMst.hpp] TownLocationMst: Response for town location master data.
- getGroupName(): "1y2JDv79".
- Data: id, pos_x, pos_y, width, height, effect_type, need_mission_id, name; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: all numeric fields 0, name empty }

[TownFacilityMst.hpp] TownFacilityMst: Response for town facility master data.
- getGroupName(): "Lh1I3dGo".
- Data: id, pos_x, pos_y, width, height, effect_type, need_mission_id, name; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: all numeric fields 0, name empty }

[SummonTicketV2UserInfo.hpp] SummonTicketV2UserInfo: Response for summon ticket V2 user info.
- getGroupName(): "a3d5d12i".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[SlotGamePictureInfo.hpp] SlotGamePictureInfo: Response for slot game picture info.
- getGroupName(): nullptr.
- Data: id, pictureName; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=0, pictureName empty }

[TownFacilityLvMst.hpp] TownFacilityLvMst: Response for town facility level master data.
- getGroupName(): "d0EkJ4TB".
- Data: id, lv, karma, release_receipe; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=0, lv=0, karma=0, release_receipe empty }

[TownLocationLvMst.hpp] TownLocationLvMst: Response for town location level master data.
- getGroupName(): "9ekQ4tZq".
- Data: id, lv, karma, release_receipe; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=0, lv=0, karma=0, release_receipe empty }

[UserArenaInfo.hpp] UserArenaInfo: Response for arena info.
- getGroupName(): "8jBJ7uKR".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserClearMissionInfo.hpp] UserClearMissionInfo: Response for cleared mission info.
- getGroupName(): "UT1SVg59".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UrlMst.hpp] UrlMst: Response for URL master data.
- getGroupName(): "At7Gny2V".
- Fields: id, officialSite, noticeUrl, contactUrl, friendRefeerUrl, appliDlUrl, appliDlAndroidUrl, famiAppSiteUrl, twitterSiteUrl, facebookSiteUrl, transferSiteUrl, appBankSiteUrl, loblSiteUrl, loblSchemaUrl, appliSommelierUrl, creditUrl, gameGiftUrl, agreementUrl, agreementOfficialUrl, legalfundSettlementUrl, specificTradeUrl, diaPossessionUrl, lobiRecHelpUrl, lobiAgreementUrl, gachaContentsUrl, multiArchiveUrl.
- SerializeFields(): Outputs fields to JSON with obfuscated keys.
{ Non-array response, default constructor: id=1, all strings empty }

[fcoRMe14] Unknown: Stray string, possibly a group name or typo.
- getGroupName(): "fcoRMe14" (assumed).
- Data: N/A.
- Mst: N/A.
{ Placeholder, unclear purpose, awaiting clarification }

[UnitExpPatternMst.hpp] UnitExpPatternMst: Response for unit exp pattern master data.
- getGroupName(): "JYFGe9y6".
- isArray(): True.
- Data: id, lv, needExp; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor not explicit, fields uninitialized }

[UserBraveMedalInfo.hpp] UserBraveMedalInfo: Response for brave medal info.
- getGroupName(): "6C0kzwM5".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserDungeonKeyInfo.hpp] UserDungeonKeyInfo: Response for dungeon key info.
- getGroupName(): "eFU7Qtb0".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserAchievementInfo.hpp] UserAchievementInfo: Response for achievement info.
- getGroupName(): "Bnc4LpM8".
- isArray(): False.
- SerializeFields(): Empty with TODO.
{ Non-array response, placeholder }

[UserItemDictionaryInfo.hpp] UserItemDictionaryInfo: Response for item dictionary info.
- getGroupName(): "bd5Rj6pN".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserFavorite.hpp] UserFavorite: Response for user favorites.
- getGroupName(): "3kcmQy7B".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserEquipItemInfo.hpp] UserEquipItemInfo: Response for equip item info.
- getGroupName(): "71U5wzhI".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserLoginCampaignInfo.hpp] UserLoginCampaignInfo: Response for login campaign info.
- getGroupName(): "3da6bd0a".
- Fields: id, currentDay, totalDays, firstForTheDay.
- SerializeFields(): Outputs fields to JSON with obfuscated keys.
{ Non-array response, default constructor: id=1, currentDay=0, totalDays=0, firstForTheDay=false }

[UserGiftInfo.hpp] UserGiftInfo: Response for gift info.
- getGroupName(): "30uygM9m".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserPartyDeckInfo.hpp] UserPartyDeckInfo: Response for party deck info.
- getGroupName(): "dX7S2Lc1".
- Data: deckType, deckNum, memberType, dispOrder, userUnitID; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: all fields 0 }

[UserLevelMst.hpp] UserLevelMst: Response for level master data.
- getGroupName(): "YDv9bJ3s".
- Data: level, exp, actionPoints, deckCost, friendCount, addFriendCount; serializes to JSON with obfuscated keys.
- Mst: Unordered_map<int, Data>, serialized by level index (throws if not found).
{ Default constructor: all fields 0 }

[UserEquipBoostItemInfo.hpp] UserEquipBoostItemInfo: Response for equip boost item info.
- getGroupName(): "nAligJSQ".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserPurchaseInfo.hpp] UserPurchaseInfo: Response for purchase info.
- getGroupName(): "4W6EhXLS".
- isArray(): False.
- SerializeFields(): Empty implementation.
{ Non-array response, placeholder }

[UserInfo.hpp] UserInfo: Response for user info.
- getGroupName(): "IKqx1Cn9".
- Fields: accountID, userID, friendID, password, handleName, contactID, tutorialStatus, tutorialEndFlag, modelChangeCount, codeExpireDate, friendInvitationFlag, earlyBirdEnd, debugMode, encryptIV, encryptedFriendID, firstDesc, dlcUrl, featureGate, serviceRequestEndPointParam.
- SerializeFields(): Outputs fields to JSON with obfuscated keys, skips empty dlcUrl/serviceRequestEndPointParam.
{ Default constructor: tutorialStatus=12, tutorialEndFlag=1, others 0/empty; Hardcoded "32k0ahkD"="773c9af44721a014c7ed" }

[UserUnitDbbInfo.hpp] UserUnitDbbInfo: Response for unit DBB (Dual Brave Burst) info.
- getGroupName(): "sxorQ3Mb".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserUnitDbbLevelInfo.hpp] UserUnitDbbLevelInfo: Response for unit DBB level info.
- getGroupName(): "tR4katob".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserUnitDictionary.hpp] UserUnitDictionary: Response for unit dictionary.
- getGroupName(): "GV81ctzR".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserUnitInfo.hpp] UserUnitInfo: Response for detailed unit info.
- getGroupName(): "4ceMWH6k".
- Data: Extensive struct with userID, unit stats (hp, atk, def, heal), skills, equipment, etc.; serializes to obfuscated JSON keys.
- Mst: Vector of Data, serialized as array.
{ Fields: userUnitID, unitID, leaderSkillID, skillID, extraSkillID, equip IDs, stats (base/add/ext/limitOver), element, etc.; Default constructor initializes to minimal values }

[VersionInfo.hpp] VersionInfo: Response for version info.
- getGroupName(): "KeC10fuL".
- Data: Id, Version, Unknown, SubVersion; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: Version=0, Unknown=0, SubVersion=0 }

[VideoAdRegion.hpp] VideoAdRegion: Response for video ad regions.
- getGroupName(): "bpD29eiQ".
- Data: id, countryCodes; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=0 }

[VideoAdsSlotgameInfo.hpp] VideoAdsSlotgameInfo: Response for video ad slot game info.
- getGroupName(): "mebW7mKD".
- Fields: gameInfo(SlotGameInfo), reelInfo(SlotGameReelInfo), pictureInfo(SlotGamePictureInfo), gameStandInfo(VideoAdsSlotGameStandInfo).
- CacheFields(): Pre-serializes gameInfo, reelInfo, pictureInfo to strings.
- SerializeFields(): Outputs cached strings and serializes gameStandInfo on-the-fly.
{ Non-array response, uses caching for efficiency }

[VideoAdsSlotGameStandInfo.hpp] VideoAdsSlotGameStandInfo: Sub-response for slot game stand info.
- getGroupName(): nullptr (non-array).
- Fields: ads_count, max_ads_count, current_bonus, max_bonus_count, ads_bonus_flag, next_day_timer.
- SerializeFields(): Outputs fields to JSON with obfuscated keys.
{ Default constructor: all fields 0 }

[UserWarehouseInfo.hpp] UserWarehouseInfo: Response for warehouse info.
- getGroupName(): "9wjrh74P".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[VideoAdInfo.hpp] VideoAdInfo: Response for video ad info.
- getGroupName(): "j129kD6r".
- Data: id, availableValue, regionId, videoEnabled, nextAvailableTimeLeft; serializes to JSON with obfuscated keys.
- Mst: Vector of Data, serialized as array.
{ Default constructor: id=0, availableValue=0, regionId=0, videoEnabled=false, nextAvailableTimeLeft=0 }

[UserReleaseInfo.hpp] UserReleaseInfo: Response for user release info.
- getGroupName(): "Dp0MjKAf".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserSummonerArmsInfo.hpp] UserSummonerArmsInfo: Response for summoner arms info.
- getGroupName(): "dhMmbm5p".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserTeamInfo.hpp] UserTeamInfo: Response for user team info.
- getGroupName(): "fEi17cnx".
- Fields: UserID, Level, Exp, MaxActionPoint, ActionPoint, MaxFightPoint, FightPoint, MaxUnitCount, AddUnitCount, DeckCost, MaxEquipSlotCount, MaxFriendCount, AddFriendCount, FriendPoint, Zel, Karma, BraveCoin, FriendMessage, WarehouseCount, AddWarehouseCount, WantGift, PresentCount, FriendAgreeCount, GiftReceiveCount, ActionRestTimer, FightRestTimer, FreeGems, ActiveDeck, SummonTicket, SlotGameFlag, RainbowCoin, BravePointsTotal, ColosseumTicket, ArenaDeckNum, ReinforcementDeck, ReinforcementDeckEx1, ReinforcementDeckEx2, PaidGems, MysteryBoxCount, CompletedTaskCount, InboxMessageCount, CurrentBravePoints.
- SerializeFields(): Outputs fields to JSON with obfuscated keys.
{ Non-array response, default constructor initializes all to 0 or empty }

[UserTeamArchive.hpp] UserTeamArchive: Response for team archive info.
- getGroupName(): "zI2tJB7R".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserTeamArenaArchive.hpp] UserTeamArenaArchive: Response for team arena archive info.
- getGroupName(): "PQ56vbkI".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserTownFacilityInfo.hpp] UserTownFacilityInfo: Response for town facility info.
- getGroupName(): "YRgx49WG".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserSummonerInfo.hpp] UserSummonerInfo: Response for summoner info.
- getGroupName(): "n5mdIUqj".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserTownLocationInfo.hpp] UserTownLocationInfo: Response for town location info.
- getGroupName(): "yj46Q2xw".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserTownLocationDetail.hpp] UserTownLocationDetail: Response for town location detail.
- getGroupName(): "s8TCo2MS".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[UserSoundInfo.hpp] UserSoundInfo: Response for sound info.
- getGroupName(): "d98mjNDc".
- Data: Empty struct with Serialize() TODO.
- Mst: Vector of Data, serialized as array.
{ Placeholder, unimplemented }

[BadgeInfo.hpp] BadgeInfo: Response for badge info (placeholder).
- getGroupName(): Unknown (not provided).
- Serialize(Json::Value&): Assumed to exist, used in BadgeInfoHandler.
{ Minimal implementation assumed }

[SlotGameInfo_Resp.hpp] SlotGameInfoR: Response for slot game info (ControlCenterEnterHandler variant).
- getGroupName(): Unknown (not provided).
- Fields: info (id, name, reelPos, useMedal, slotHelpUrl, slotImage), pictures (SlotGamePictureInfo).
- Serialize(Json::Value&): Assumed to exist, used in ControlCenterEnterHandler.
{ Hardcoded data in handler }

[ReinforcementInfo.hpp] ReinforcementInfo: Response for friend/reinforcement info.
- getGroupName(): Unknown (not provided).
- Serialize(Json::Value&): Assumed to exist, used in FriendGetHandler.
{ Minimal implementation assumed }

[SignalKey.hpp] SignalKey: Response for signal key.
- getGroupName(): Unknown (not provided).
- Fields: key (string).
- Serialize(Json::Value&): Exists, used in multiple handlers (e.g., "fZnLr4t9", "axhp8Sin", "C7vnXA5T").
{ Simple key wrapper }

[ChallengeArenaUserInfo.hpp] ChallengeArenaUserInfo: Response for challenge arena user info.
- getGroupName(): Unknown (not provided).
- Fields: unkstr, unkstr2, leagueId.
- Serialize(Json::Value&): Exists, used in InitializeHandler.
{ Hardcoded: unkstr="n9ZMPC0t", unkstr2="F", leagueId=1 }

[DailyTaskPrizeMst.hpp] DailyTaskPrizeMst: Response for daily task prize master data.
- getGroupName(): Unknown (not provided).
- Data: taskId, taskTitle, taskDesc, bravePointCost, currentClaimCount, isMileStonePrize, maxClaimCount, presentType, targetCount, targetId, targetParam, timeLimit.
- Mst: Vector of Data, serialized as array.
{ Populated from MstConfig.DailyTaskConfig().GetTaskPrizes() }

[DailyTaskBonusMst.hpp] DailyTaskBonusMst: Response for daily task bonus master data.
- getGroupName(): Unknown (not provided).
- Data: bonusBravePoints.
- Mst: Vector of Data, serialized as array.
{ Populated from MstConfig.DailyTaskConfig().GetTaskBonusTables() }

[NoticeInfo.hpp] NoticeInfo: Response for notice info.
- getGroupName(): Unknown (not provided).
- Fields: Unknown, assumed to include URL (e.g., "http://127.0.0.1:9960/bf/web/html/notice/index.php").
- Serialize(Json::Value&): Exists, used in DeckEditHandler.
{ Minimal implementation assumed }

[GuildInfo.hpp] GuildInfo: Response for guild info.
- getGroupName(): Unknown (not provided).
- Serialize(Json::Value&): Exists, used in InitializeHandler.
{ Minimal implementation assumed }

[DailyTaskMst.hpp] DailyTaskMst: Response for daily task master data.
- getGroupName(): Unknown (not provided).
- Data: typeKey, typeTitle, typeDescription, taskCount, taskBravePoints.
- Mst: Vector of Data, serialized as array.
{ Hardcoded tasks: AV("Arena Victory"), VV("Vortex Venturer"), CM("Craftsman") }

[DailyLoginRewardsUserInfo.hpp] DailyLoginRewardsUserInfo: Response for daily login rewards user info.
- getGroupName(): Unknown (not provided).
- Serialize(Json::Value&): Exists, used in InitializeHandler.
{ Minimal implementation, TODO for configurability }

[BannerInfoMst.hpp] BannerInfoMst: Response for banner info master data (placeholder).
- getGroupName(): Unknown (not provided).
- Serialize(Json::Value&): Assumed part of MstConfig.CopyInitializeMstTo().
{ Not provided, assumed minimal }

[NoticeListMst.hpp] NoticeListMst: Response for notice list master data (placeholder).
- getGroupName(): Unknown (not provided).
- Serialize(Json::Value&): Assumed part of MstConfig.CopyInitializeMstTo().
{ Not provided, assumed minimal }

[NpcMst.hpp] NpcMst: Response for NPC master data (placeholder).
- getGroupName(): Unknown (not provided).
- Serialize(Json::Value&): Assumed part of MstConfig.CopyInitializeMstTo().
{ Not provided, assumed minimal }

[SummonerJournalUserInfo.hpp] SummonerJournalUserInfo: Response for summoner journal user info.
- getGroupName(): Unknown (not provided).
- Fields: userId.
- Serialize(Json::Value&): Exists, used in InitializeHandler.
{ Minimal implementation, userId from UserInfo }

---

## Encryption and Utilities

[BfCrypt.hpp/.cpp] BfCrypt: Encrypts/decrypts JSON data.
- CryptSREE(Json::Value): AES-CBC with SREE_KEY/SREE_IV, base64-encodes.
{ SREE_KEY: "7410958164354871", SREE_IV: "Bfw4encrypedPass" }
- CryptGME(Json::Value, string): AES-ECB with dynamic key (padded to 16 bytes), base64-encodes.
- DecryptGME(string, string, Json::Value&): Decrypts base64 input with dynamic key, parses to JSON.
{ Uses CryptoPP, logs errors }

[Utils.hpp/.cpp] Utils: Misc utilities.
- DumpInfoToDrogon(HttpRequestPtr, string): Logs request details.
- GetDrogonBindHostname(): Returns "127.0.0.1:9960".
- GetDrogonHttpBindHostname(): Returns "http://127.0.0.1:9960/".
- RandomUserID(), RandomAccountID(): 8-char random alphanumeric.
- AppendJsonReqToFile(Json::Value, string), AppendJsonResToFile(Json::Value, string): Logs JSON to files with timestamp.
- AddMissingDlcFile(string): Appends to DLC 404 log.
{ Logging depends on LogConfig.Enable }

---

## Notes
- Client: Decompiled from libgame.so, patched to loopback, libcurl.dll points to 127.0.0.1:9960.
- Resources: Units, missions, items, synergy, extra skills identified in decomp.
- Current Issue: Unit inventory crashes client at large increments (e.g., 1450->1500).
