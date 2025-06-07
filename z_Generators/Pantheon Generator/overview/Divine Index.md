```dataview
table
  tier      as "Tier",
  domain    as "Domain",
  alignment as "Alignment",
  pantheon  as "Pantheon",
  holy_day  as "Holy Day"
from "2-World/Cosmology/Deities"
where tier
sort pantheon asc, tier asc, name asc
```


```dataviewjs
// 1) Load all pages under your Deities folder that have both 'tier' and 'pantheon' keys
const allDeities = dv
  .pages('"2-World/Cosmology/Deities"')
  .where(p => p.tier && p.pantheon);

// 2) Group by pantheon → returns an array of { key: pantheonName, rows: DataArray }
const byPantheon = allDeities.groupBy(p => p.pantheon);

// 3) Define a “canonical” tier order (adjust if you have custom tiers)
const tierOrder = [
  "True God",
  "Lesser God",
  "Demigod",
  "Angel",
  "Saint",
  "Champion",
  "False God",
  "Shattered God"
];

// 4) For each pantheon, render a header, count, then a single‐row table
for (const pantheonGroup of byPantheon) {
  const speciesName = pantheonGroup.key;
  const pagesInPantheon = pantheonGroup.rows; // DataArray of all gods in this pantheon

  // Show the pantheon header and total number of deities
  dv.header(2, `${speciesName} Pantheon`);
  dv.paragraph(`**Total Deities:** ${pagesInPantheon.length}`);

  // Next: group THIS pantheon’s deities by tier
  // .groupBy on a DataArray returns an array of objects { key: tierName, rows: DataArray }
  const byTier = pagesInPantheon.groupBy(p => p.tier);

  // Build a map: tierName → array of page objects
  const tierMap = {};
  for (const tierGroup of byTier) {
    tierMap[tierGroup.key] = tierGroup.rows;
  }

  // 5) Build the Markdown table header (column titles = each tier)
  const headerRow = "|" + tierOrder.map(t => ` ${t} `).join("|") + "|";
  const dividerRow = "|" + tierOrder.map(_ => "---").join("|") + "|";

  // 6) Populate a single row: for each tier, join all deity names (as links)
  let contentRow = "|";
  for (const tierName of tierOrder) {
    if (tierMap[tierName]) {
      // Extract each DataArray item’s file.name (basename) and wrap in [[ ]]
      const links = tierMap[tierName]
        .map(d => `[[${d.file.name}]]`)
        .sort(); // optional: sort alphabetically
      contentRow += ` ${links.join(", ")} |`;
    } else {
      contentRow += "  |"; // empty cell if no deities in that tier
    }
  }

  // 7) Emit the raw Markdown table so Dataview will render it
  dv.paragraph(headerRow + "\n" + dividerRow + "\n" + contentRow);
}
```

