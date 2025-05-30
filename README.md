# Pantheon Generator for Obsidian

A randomized deity generator designed for worldbuilders, game masters, and TTRPG storytellers. This script dynamically assembles an original pantheon with rich detail using Templater inside [Obsidian](https://obsidian.md/).

---

## Overview

This generator creates deities across a structured divine hierarchy, assigning each one:

- **Name & Epithet**
- **Divine Tier** (e.g., True God, Demigod, Angel, etc.)
- **Domain, Alignment, and Pantheon**
- **Mythic Aspects** (e.g., tenets, holy days, myths)
- **Relationships** (e.g., parent of, rival to, created by)

It also outputs a visual **family tree** using Mermaid and a **collapsible summary table** to track the generated pantheon.

---

## Divine Hierarchy

This project uses a custom cosmological framework inspired by comparative mythology. The ten tiers are:

1. **Prime Source** – The unknowable origin
2. **True Gods** – Independent realm-holding deities
3. **Lesser Gods** – Regional or narrower in domain
4. **Demigods** – Mortal-divine hybrids
5. **Angels** – Obedient, single-purpose servants
6. **Saints** – Mortals exalted through martyrdom
7. **Champions** – Mission-driven mortal agents
8. **False Gods** – Manifested from mass belief
9. **Shattered Gods** – Dead, forgotten, or looted deities
10. **Conceptualism** – Worship of abstract forces

See `Hierarchy.md` for full descriptions and examples.

---

## 🛠 How It Works

### Tools Required:
- [Obsidian](https://obsidian.md/)
- [ITS Theme](https://github.com/SlRvb/Obsidian--ITS-Theme)
- [Templater Plugin](https://github.com/SilentVoid13/Templater)

### Input Data:
Each `.md` file in `z_Generators/Pantheon Generator/data` contains a themed list (e.g., `colors.md`, `myths.md`, `tenets.md`) used to fill in random values.

### Core Script:
The logic lives in `pantheon-generator.md`, which:
- Loads the word banks
- Generates 20–30 gods based on tier distributions
- Builds callouts, relationships, and diagrams dynamically

---

## Output Example

- Collapsible family tree (`graph TD`)
	- Rendered using [Mermaid.js](https://mermaid.js.org/)
- Scrollable markdown summary table
- Tier-specific callouts with icon and styling
- Randomly assigned relationships and symbols

![FamilyTree.png](example/FamilyTree.png)
![SummaryTable.png](example/SummaryTable.png)
![PantheonList.png](example/PantheonList.png)
![GodCallout.png](example/GodCallout.png)



---

## License

This project is open source under the **MIT License**.  
Creative text elements (e.g., myths, tenets, names) are shared under **CC BY-NC 4.0**.  
Feel free to fork, adapt, and expand—just don’t sell it without asking.

---

## Credits

Made by [@Gulgrot](https://github.com/Gulgrot) as a hobby project.  
Inspired by comparative theology, storytelling tropes, and the joy of procedural generation.

---

## Related

- [`pantheon-generator.md`](./pantheon-generator.md) – Main script
- [`Hierarchy.md`](./Hierarchy.md) – Divine structure
- `/data/` – Input banks for generator

