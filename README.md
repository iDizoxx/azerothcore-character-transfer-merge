# azerothcore-character-transfer-merge
Method for merging or transferring an AzerothCore database from one server to another.


# AzerothCore Database Merge & Transfer Guide

## Introduction
This guide provides a step-by-step method for merging or transferring an AzerothCore database from one server to another. The process ensures that account, character, and other relevant data are correctly incremented to avoid ID conflicts. 

### Logic for Merging
- Set the logic parameters for merging based on either an incremental method or a predefined set of values.
- Identify the highest account ID and highest character ID in the existing databases to determine appropriate increments.

**Important:** The server must be offline while preparing data for migration!

---

## Steps for Database Merge / Transfer

### Step 1: Prepare the Databases
1. **Turn off your AzerothCore server.**
2. **Backup your databases:**
   - `acore_auth` (accounts database)
   - `acore_characters` (character data database)

### Step 2: Choose the Incrementing Method

#### **Method 1: Dynamic Increment Based on Highest ID**
1. Retrieve the highest **account ID** from `acore_auth.account (column: id)`.
2. Retrieve the highest **character ID** from `acore_characters.characters (column: guid)`.
3. Update database values accordingly.

#### **Method 2: Fixed Increment Values**
Use predefined increments to avoid conflicts:
- Account ID increment: **99,000**
- Character ID increment: **88,000**
- Arena Team ID increment: **77,000**
- Pet ID increment: **66,000**
- Guild ID increment: **55,000**

**Important:** Ensure the predefined values are higher than the highest existing IDs in your databases. You can check this using the following SQL queries:
```sql
SELECT MAX(id) FROM acore_auth.account;
SELECT MAX(guid) FROM acore_characters.characters;
SELECT MAX(id) FROM acore_characters.character_pet;
SELECT MAX(guildid) FROM acore_characters.guild;
```

### Step 3: Apply Increment Changes (SQL Script)
If you choose **Method 2**, apply the following SQL script:

```sql
-- Update in database acore_auth
UPDATE acore_auth.account SET id = id + 99000;

-- Update in database acore_characters
UPDATE acore_characters.characters SET account = account + 99000;
UPDATE acore_characters.characters SET guid = guid + 88000;

UPDATE acore_characters.account_data SET accountId = accountId + 99000;
UPDATE acore_characters.account_instance_times SET accountId = accountId + 99000;
UPDATE acore_characters.account_tutorial SET accountId = accountId + 99000;

UPDATE acore_characters.arena_team SET arenaTeamId = arenaTeamId + 77000;
UPDATE acore_characters.arena_team SET captainGuid = captainGuid + 88000;
UPDATE acore_characters.arena_team_member SET arenaTeamId = arenaTeamId + 77000;
UPDATE acore_characters.arena_team_member SET guid = guid + 88000;

UPDATE acore_characters.battleground_deserters SET guid = guid + 88000;
UPDATE acore_characters.character_achievement SET guid = guid + 88000;
UPDATE acore_characters.character_equipmentsets SET guid = guid + 88000;
UPDATE acore_characters.character_glyphs SET guid = guid + 88000;
UPDATE acore_characters.character_inventory SET guid = guid + 88000;

UPDATE acore_characters.character_pet SET id = id + 66000;
UPDATE acore_characters.character_pet SET owner = owner + 88000;

UPDATE acore_characters.character_queststatus SET guid = guid + 88000;
UPDATE acore_characters.character_social SET guid = guid + 88000;
UPDATE acore_characters.character_social SET friend = friend + 88000;
UPDATE acore_characters.character_talent SET guid = guid + 88000;
UPDATE acore_characters.corpse SET guid = guid + 88000;

UPDATE acore_characters.guild SET guildid = guildid + 55000;
UPDATE acore_characters.guild SET leaderguid = leaderguid + 88000;
UPDATE acore_characters.guild_bank_item SET guildid = guildid + 55000;

UPDATE acore_characters.pet_aura SET guid = guid + 66000;
UPDATE acore_characters.pet_spell SET guid = guid + 66000;
```

### Step 4: Export Required Tables
To export only the necessary tables for migration, use the following commands:

#### Export `acore_auth` Tables
```sh
mysqldump -u root -p acore_auth account > acore_auth_backup.sql
```

#### Export `acore_characters` Tables
```sh
mysqldump -u root -p acore_characters \
characters account_data account_instance_times account_tutorial \
arena_team arena_team_member battleground_deserters character_achievement \
character_equipmentsets character_glyphs character_inventory \
character_pet character_queststatus character_social character_talent \
guild guild_bank_item pet_aura pet_spell > acore_characters_backup.sql
```

### Step 5: Import the Modified Data into the New Server
1. Restore the modified `acore_auth` and `acore_characters` databases into the new server.
2. Start the AzerothCore server and verify the migration.

---

## Additional Notes
- Ensure that every new account ID continues from the highest existing ID in the database.
- If using **Method 1**, dynamically adjust all relevant tables based on the maximum existing values.
- Refer to the official AzerothCore database documentation for further details: [AzerothCore Characters Database](https://www.azerothcore.org/wiki/characters)

Following these steps will ensure a successful and conflict-free migration of accounts and characters between AzerothCore servers.

---

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## Contact
For any questions or issues, feel free to open a discussion in the repository.

