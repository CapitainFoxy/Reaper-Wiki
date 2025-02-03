# Üdvözöllek a Reaper Discord Bot fejlesztői oldalán!
Itt is jelezheted felénk ha valami problémát találtál a bottal kapcsolatban!

Amennyiben szeretnél egy új funkcót azt is itt jelezheted felénk!

Illetve itt megtalálsz mindent, amire egy parancs fejlesztése során szükséged lehet!

A fejlesztéshez jelenleg **Discord.js ^14.0.0** használandó!

# Tartalom

Cím | Leírás
------------ | -------------
[Új funkció vagy hiba jelentése](#issue-example) | Új funckió vagy hiba jelentésének menete és szükséges infók
[Parancs példa](#command-example) | Parancs létrehozásához szükséges infók
[Parancs válaszüzenet](#command-reply-example) | Parancsokra való válaszüzenet küldése
[Nyelvi fájlok](#command-translate-example) | Parancsok nyelvi fájlainak létrehozása és használata
[Adatbázis](#database) | Adatbázis használat
[ESLint](#eslint) | Kód formázás és ESLint használat


# Új funkció vagy hiba jelentése <a name="issue-example"></a>

Amennyiben ötleted van fejlesztésre vagy hibát találtál, kérlek jelezd felénk itt Githubon [ide kattintva](https://github.com/CapitainFoxy/Reaper-Wiki/issues/new/choose)!

# Parancs példa <a name="command-example"></a>

Parancsokat commands mappán belül mindig a hozzá tartozó kategóia néven szereplő mappában kell elhelyezni. Ezért például egy ping parancs mivel általános parancs, így a General mappában található, tehát az elérhetőség: '/commands/General/ping.js'

A parancsok fejlesztésének nulladik lépéseként mindig kötelező a következő példában bemutatott alap használata:

```js
const Command = require('../../database/Command.js')

class Ping extends Command { // class neve minden esetben a parancs neve nagybetűvel fontos ellenőrizni, hogy nincs-e már ilyen parancs!
  constructor (client) {
    super(client, {
      name: 'examplePingCommand', // parancs neve
      dirname: __dirname,
      enabled: true, // Ha szükséges lehetőség van parancs letiltására, amíg fejlesztés alatt van!
      guildOnly: true,  // Amennyiben olyan adatokat kell használni, ami csak szerveren elérhető, úgy true
      memberPermissions: [], // Szükség lehet jogosultsághoz kötni a parancsot, ilyenkor a memberPermissions-ben kell ezeket meghatározni pl: memberPermissions: ['ADMINISTRATOR'],
      botPermissions: ['SEND_MESSAGES', 'MANAGE_ROLES'], // Parancs használatához szükséges bot jogosultságok
      nsfw: false, // Amennyiben bármilyen jellegű nsfw tartalmat tartalmazhat a parancs, akkor kötelező true értéket megadni!
      ownerOnly: false, // Csak bot fejlesztő használhatja a parancsot true érték esetében!
      cooldown: 1000, // Parancs kétszeri beírása között kötelező időköz ms-ben megadva
      //slash command options további infók: https://discord.com/developers/docs/interactions/application-commands
      options: [
        {
          name: 'exampleSlashOptions',
          description: 'Description',
          type: 3,
          required: true
        }
      ]

    })
  }
  //Perjeles parancs esetén:
  async interaction (interaction) {
    await interaction.deferReply() // Mivel időbe telhet míg a bot válaszol, ezért kötelezően használandó a DeferReply
    // Szükség esetén használandó try catch azonban nem szükséges, mivel alapból van hibakezelés a parancsoknál!
    try {

    } catch (e) {
      console.error(e)
      return client.createSlashCommandError(interaction, e)
    }
  }
  
  // amennyiben context menü parancsot szeretnél:
  async context(interaction) {
    
  }
}

module.exports = Ping

```

## Parancs válaszüzenet <a name="command-reply-example"></a>

Több lehetőség is van válaszüzenet küldésére:

```js
// sikeres üzenet küldésekor:
return interaction.success('translatedir/translatefilename:CONTENT', {data: dataForTranslate})
// Sikertelen esetében használandó (pl.: nem tartózkodik voice szobában):
return interaction.error('translatedir/translatefilename:CONTENT', {data: dataForTranslate})
return this.client.createSlashError(interaction, `üzenet`)

// Amennyiben szükséges, hogy az üzenetet csak a parancs használója láthatja, használható az { ephemeral: true } ebben az esetben a success és az error-t lehet csak használni a következő módon:
return interaction.success('translatedir/translatefilename:CONTENT', {data: dataForTranslate}, { ephemeral: true })
return interaction.error('translatedir/translatefilename:CONTENT', {data: dataForTranslate}, { ephemeral: true })
```

# Nyelvi fájlok <a name="command-translate-example"></a>

A Reaper Discord bot jelenleg Magyar nyelven készül, azonban később elképzelhető, hogy igény szerint több nyelven is elérhető lesz. Ezért nyelvi fájlok használata kötelező!

## Használat:

**Nyelvi fájl:**

**DESCRIPTION**-nek tartalmaznia kell egy rövid parancs leírást, amely tömören összefoglalja mire lehet használni az adott parancsot. Kitöltése **kötelező**!

**USAGE** részben meg kell határozni a parancs használatát, ide be kell írni azt is ha valami paramétert kell megadni a használat során. Amennyiben a paramétert kötelező megadni úgy a '<>' jelek közé kell írni a paramétert, amennyiben viszont csak opcionális, abban az esetben a '[]' használata szükséges. Kitöltése **kötelező** mivel ezeket használja a help parancs!

**CONTENT** rész csak opcionális és szükség esetén át is lehet nevezni és lehet változókat is alkalmazni pl: {{vétozó neve}}!
A következőben egy példa ping parancs látható és hozzá a nyelvi fájl:


```json
{
  "DESCRIPTION": "Bot pingjének megtekintése",
  "USAGE": "ping",
  "CONTENT": "Pong! Az én pingem `{{ping}}` ms."
}
```

```javascript
const dataForTranslate = Math.round(this.client.ws.ping)
return interaction.success('translatedir/translatefilename:CONTENT', { ping: dataForTranslate })
```

# Adatbázis <a name="database"></a>
Lehetőséged van adatokat menteni és betöltetni MongoDB adatbázisból mongoose használatával egy-egy parancs használata során.

Mongoose jelenlegi verzió: **mongoose: ^6.1.4**


## Példa:

```javascript
// Egy szerver adatainak betöltése:
const guildData = await this.client.findOrCreateGuild({ id: interaction.guild.id })
// Adat mentése:
guildData.plugins.example = 'teszt'
guildData.markModified('plugins.example')
guildData.save()

// Felhasználó
const userData = await this.client.findOrCreateUser({ id: interaction.user.id })
// Szerver felhasználó
const memberData = await this.client.findOrCreateMember({ id: interaction.user.id })
```

# ESLint <a name="eslint"></a>

ESLint használata **kötelező**!

Jelenleg általam használt verzió: **eslint ^7.32.0**

Lehetőség szerint az elkészült kód **ne tartalmazzon hibás sorokat** és mielőtt elküldenéd **teszteld a működését**!

## Beállítása:
```json
"eslintConfig": {
  "env": {
    "commonjs": true,
    "es6": true,
    "node": true
  },
  "extends": "eslint:recommended",
  "globals": {
    "Atomics": "readonly",
    "SharedArrayBuffer": "readonly"
  },
  "parserOptions": {
    "ecmaVersion": 2020
  },
  "rules": {
    "prefer-const": [
      "error"
    ],
    "indent": [
      "error",
      "space",
      {
        "SwitchCase": 1
      }
    ],
    "quotes": [
      "error",
      "double"
    ],
    "semi": [
      "error",
      "always"
    ],
    "linebreak-style": 0,
    "require-atomic-updates": 0
  }
}
```
