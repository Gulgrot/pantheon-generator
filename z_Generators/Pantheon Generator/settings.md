---
selectedPantheon: Random
amtTrueGod: 15
amtLesserGod: 12
amtDemigod: 10
amtAngel: 8
amtSaint: 8
amtChampion: 5
amtFalseGod: 14
amtShatteredGod: 9
selectedDeityCount: Reference Values Below
---
>[!metadata]+ Deity Settings
>
>
>Pantheon: `INPUT[inlineSelect(
> option(Random),
> option(Aasimar),
> option(Dragonborn),
> option(Dwarf),
> option(High Elf),
> option(Wood Elf),
> option(Drow),
> option(Gnome),
> option(Goblin),
> option(Goliath),
> option(Halfling),
> option(Human),
> option(Orc),
> option(Tiefling)
>):selectedPantheon]`
>
>Deity Count: `INPUT[inlineSelect(
> option(Random),
> option(Balanced),
> option(Reference Values Below)
> ):selectedDeityCount]`
> 
>True God: `INPUT[number(defaultValue(10)):amtTrueGod]`
>Lesser God: `INPUT[number(defaultValue(5)):amtLesserGod]`
>Demigod: `INPUT[number(defaultValue(2)):amtDemigod]`
>Angel: `INPUT[number(defaultValue(4)):amtAngel]`
>Saint: `INPUT[number(defaultValue(4)):amtSaint]`
>Champion: `INPUT[number(defaultValue(2)):amtChampion]`
>False God: `INPUT[number(defaultValue(10)):amtFalseGod]`
>Shattered God: `INPUT[number(defaultValue(10)):amtShatteredGod]`

```meta-bind-button
style: primary
label: Generate Pantheon
actions:
  - type: templaterCreateNote
    templateFile: "z_Templates/genPantheon.md"
    folderPath: 2-World/Cosmology/Deities
    fileName: "Generated Overview"
```