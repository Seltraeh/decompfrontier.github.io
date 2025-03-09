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
