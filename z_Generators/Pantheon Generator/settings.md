---
selectedPantheon: Orc
amtTrueGod: 20
amtLesserGod: 20
amtDemigod: 20
amtAngel: 20
amtSaint: 20
amtChampion: 20
amtFalseGod: 20
amtShatteredGod: 20
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
    folderPath: 2-World/Cosmology
    fileName: "Random Generated Pantheon"
```