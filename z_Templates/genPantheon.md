<%*
/*
  =====================
  Section: Utility Functions
  =====================
*/

// Load values from a file, trimming and filtering empty lines
const loadValues = async (filepath) => {
  const file = app.vault.getAbstractFileByPath(filepath);
  if (!file) return [];
  const content = await app.vault.read(file);
  return content
    .split('\n')
    .map(line => line.trim())
    .filter(line => line.length > 0);
};

// Pick a random element from an array
const pick = (arr) => arr[Math.floor(Math.random() * arr.length)];

// Pick multiple unique random elements from an array
const pickMultiple = (arr, count) => {
  const result = [];
  const available = [...arr];
  for (let i = 0; i < count && available.length > 0; i++) {
    const idx = Math.floor(Math.random() * available.length);
    result.push(available.splice(idx, 1)[0]);
  }
  return result;
};

// Simple pluralization
const pluralize = (word) => {
  if (word.endsWith("s")) return word;
  if (word.endsWith("y")) return word.slice(0, -1) + "ies";
  return word + "s";
};

// Conjugate a verb to "s" or "er" form
const conjugateVerb = (verb, form) => {
  if (form === "s") {
    if (verb.endsWith("y") && !"aeiou".includes(verb[verb.length - 2])) {
      return verb.slice(0, -1) + "ies";
    } else if (
      ["s", "x", "z"].includes(verb.slice(-1)) || /ch$|sh$/.test(verb)
    ) {
      return verb + "es";
    } else {
      return verb + "s";
    }
  }
  if (form === "er") {
    if (verb.endsWith("e")) return verb + "r";
    return verb + "er";
  }
  return verb;
};

// Capitalize first letter
const capitalize = (str) => str.charAt(0).toUpperCase() + str.slice(1);

// Title-case each word, handling hyphens
const capitalizeTitle = (str) =>
  str
    .split(" ")
    .map((word) =>
      word.includes("-")
        ? word
            .split("-")
            .map((w) => capitalize(w))
            .join("-")
        : capitalize(word)
    )
    .join(" ");

/*
  =====================
  Section: Data Loading
  =====================
*/

// Load arrays of values from various data files in parallel
const [
	adjectives,
	adverbs,
	nouns,
	verbs,
	alignments,
	ambitions,
	aspects,
	colors,
	createdBy,
	domains,
	epithets,
	hierarchy,
	holyDays,
	loyalty,
	martyrdom,
	missions,
	myths,
	pantheons,
	patrons,
	realms,
	relationships,
	ruins,
	symbols,
	tenets
] = await Promise.all([
  loadValues("z_Generators/Pantheon Generator/data/language/adjectives.md"),
  loadValues("z_Generators/Pantheon Generator/data/language/adverbs.md"),
  loadValues("z_Generators/Pantheon Generator/data/language/nouns.md"),
  loadValues("z_Generators/Pantheon Generator/data/language/verbs.md"),
  loadValues("z_Generators/Pantheon Generator/data/alignments.md"),
  loadValues("z_Generators/Pantheon Generator/data/ambitions.md"),
  loadValues("z_Generators/Pantheon Generator/data/aspects.md"),
  loadValues("z_Generators/Pantheon Generator/data/colors.md"),
  loadValues("z_Generators/Pantheon Generator/data/created-by.md"),
  loadValues("z_Generators/Pantheon Generator/data/domains.md"),
  loadValues("z_Generators/Pantheon Generator/data/epithets.md"),
  loadValues("z_Generators/Pantheon Generator/data/hierarchy.md"),
  loadValues("z_Generators/Pantheon Generator/data/holy-days.md"),
  loadValues("z_Generators/Pantheon Generator/data/loyalty.md"),
  loadValues("z_Generators/Pantheon Generator/data/martyrdom.md"),
  loadValues("z_Generators/Pantheon Generator/data/missions.md"),
  loadValues("z_Generators/Pantheon Generator/data/myths.md"),
  loadValues("z_Generators/Pantheon Generator/data/pantheons.md"),
  loadValues("z_Generators/Pantheon Generator/data/patrons.md"),
  loadValues("z_Generators/Pantheon Generator/data/realms.md"),
  loadValues("z_Generators/Pantheon Generator/data/relationships.md"),
  loadValues("z_Generators/Pantheon Generator/data/ruins.md"),
  loadValues("z_Generators/Pantheon Generator/data/symbols.md"),
  loadValues("z_Generators/Pantheon Generator/data/tenets.md")
]);

// Default word bank for template rendering
const DEFAULT_WORD_BANK = {
	nounList: nouns,
	adjList: adjectives,
	advList: adverbs,
	verbList: verbs,
	colorList: colors
};

// Render a template string with placeholders using provided word lists
const renderTemplate = (template, wordBank = DEFAULT_WORD_BANK) => {
  let rendered = (template || "{{adj}} {{noun}}")
	.replace(/{{adj}}/g, () => pick(wordBank.adjList))
    .replace(/{{adverb}}/g, () => pick(wordBank.advList))
    .replace(/{{color}}/g, () => pick(wordBank.colorList))
    .replace(/{{noun}}/g, () => pick(wordBank.nounList))
    .replace(/{{verb}}/g, () => pick(wordBank.verbList))
    .replace(/{{verb}}er/g, () => conjugateVerb(pick(wordBank.verbList), "er"))
    .replace(/{{verb}}s/g, () => conjugateVerb(pick(wordBank.verbList), "s"))
    .replace(/{{verb_cap}}/g, () => capitalize(pick(wordBank.verbList)))
    .replace(/{{verb_cap}}er/g, () => capitalize(conjugateVerb(pick(wordBank.verbList), "er")))
    .replace(/{{verb_cap}}s/g, () => capitalize(conjugateVerb(pick(wordBank.verbList), "s")))
    .replace(/{{a}} (?=[a-zA-Z])/g, (match, offset, str) => {
      const nextWordMatch = str.slice(offset + match.length).match(/^\w+/);
      const nextWord = nextWordMatch ? nextWordMatch[0].toLowerCase() : "";
      return /^[aeiou]/.test(nextWord) ? "an " : "a ";
    });

  return capitalize(rendered);
};

/*
  ====================================
  Section: Frontmatter and Settings
  ====================================
*/

// Load frontmatter (YAML) from settings.md
const loadFrontmatter = async (filepath) => {
  const file = app.vault.getAbstractFileByPath(filepath);
  if (!file) return {};
  const content = await app.vault.read(file);
  const match = content.match(/^---\n([\s\S]*?)\n---/);
  if (!match) return {};

  const numericKeys = [
    "amtTrueGod",
    "amtLesserGod",
    "amtDemigod",
    "amtAngel",
    "amtSaint",
    "amtChampion",
    "amtFalseGod",
    "amtShatteredGod"
  ];

  return Object.fromEntries(
    match[1]
      .split('\n')
      .map((line) => line.trim())
      .filter((line) => line.includes(':'))
      .map((line) => {
        const [key, ...rest] = line.split(':');
        const value = rest.join(':').trim();
        return [
          key.trim(),
          numericKeys.includes(key.trim()) ? parseInt(value) : value,
        ];
      })
  );
};

const pantheonSettings = await loadFrontmatter(
  "z_Generators/Pantheon Generator/settings.md"
);

// Determine pantheon (random or selected)
let pantheon;
if (!pantheonSettings.selectedPantheon || pantheonSettings.selectedPantheon === "Random") {
  pantheon = pick(pantheons);
} else {
  pantheon = pantheonSettings.selectedPantheon;
}

const pantheonSlug = pantheon.toLowerCase().replace(/ /g, "-");
const pantheonNamePath = `z_Generators/Pantheon Generator/data/names/names-${pantheonSlug}.md`;
const names = await loadValues(pantheonNamePath);

/*
  ===========================
  Section: Name Management
  ===========================
*/

// Get all existing deity names under a root folder
const getAllDeityNames = async (rootPath) => {
  const allFiles = app.vault.getFiles();
  const deityFiles = allFiles.filter(
    (f) => f.path.startsWith(rootPath) && f.path.endsWith(".md")
  );
  return new Set(deityFiles.map((f) => f.basename.trim()));
};

const existingNames = await getAllDeityNames(
  "2-World/Cosmology/Deities/"
);
const usedNames = new Set();
const availableNames = names.filter((name) => !existingNames.has(name));

// Pick a unique deity name, ensuring no repeats
const pickUniqueName = () => {
  if (availableNames.length === 0) {
    throw new Error("Ran out of unique names!");
  }
  const idx = Math.floor(Math.random() * availableNames.length);
  const name = availableNames.splice(idx, 1)[0];
  usedNames.add(name);
  return name;
};

/*
  ==========================
  Section: Deity Generation
  ==========================
*/

// Generate a single god object with properties based on tier
const generateGod = (tier) => {
  const baseGod = {
    name: pickUniqueName(),
    tier: tier,
    epithet: capitalizeTitle(renderTemplate(pick(epithets))),
    domain: pick(domains),
    alignment: pick(alignments),
    aspects: pickMultiple(aspects, Math.floor(Math.random() * 3) + 1),
    colors: pickMultiple(colors, Math.floor(Math.random() * 3) + 1),
    patrons: pickMultiple(patrons, Math.floor(Math.random() * 2) + 1),
    holyDay: pick(holyDays),
    tenet: pick(tenets),
    myth: pick(myths),
    isGoddess: Math.random() < 0.5,
    mainSymbol: renderTemplate(pick(symbols)),
    subSymbols: pickMultiple(nouns, Math.floor(Math.random() * 2) + 1).map((noun) => capitalize(pluralize(noun)))
  };
  enrichByTier(baseGod);
  return baseGod;
};

// Add or adjust properties specific to each tier
const enrichByTier = (god) => {
  switch (god.tier) {
    case "True God":
      god.realm = capitalizeTitle(renderTemplate(pick(realms)));
      god.aspects = pickMultiple(aspects, 3);
      god.subSymbols = pickMultiple(nouns, 2).map((noun) => capitalize(pluralize(noun)));
      break;
    case "Demigod":
      god.ambition = pick(ambitions);
      god.loyalty = pick(loyalty);
      break;
    case "Angel":
      god.serves = pick(names);
      god.patrons = ["-"];
      god.holyDay = "-";
      god.myth = "-";
      break;
    case "Saint":
      god.aspects = ["-"];
      god.martyrdom = pick(martyrdom);
      break;
    case "Champion":
      god.aspects = ["-"];
      god.patrons = ["-"];
      god.holyDay = "-";
      god.serves = pick(names);
      god.mission = pick(missions);
      break;
    case "False God":
      god.createdBy = pick(createdBy);
      break;
    case "Shattered God":
      god.holyDay = "-";
      god.martyrdom = pick(martyrdom);
      god.ruinSite = capitalizeTitle(renderTemplate(pick(ruins)));
      break;
  }
};

// Determine number of deities per tier based on settings
let tierCounts;
if (!pantheonSettings.selectedDeityCount || pantheonSettings.selectedDeityCount === "Random") {
  // Random count between 0 and 20 for each tier
  tierCounts = Object.fromEntries(
    [
      "True God",
      "Lesser God",
      "Demigod",
      "Angel",
      "Saint",
      "Champion",
      "False God",
      "Shattered God"
    ].map((tier) => [tier, Math.floor(Math.random() * 21)])
  );
} else if (pantheonSettings.selectedDeityCount === "Balanced") {
  // Predefined balanced ranges
  tierCounts = {
    "True God": Math.floor(Math.random() * 5) + 8, // 8–12
    "Lesser God": Math.floor(Math.random() * 3) + 4, // 4–6
    "Demigod": Math.floor(Math.random() * 2) + 1, // 1–2
    "Angel": Math.floor(Math.random() * 3) + 2, // 2–4
    "Saint": Math.floor(Math.random() * 3) + 2, // 2–4
    "Champion": Math.floor(Math.random() * 2) + 1, // 1–2
    "False God": Math.floor(Math.random() * 2) + 1, // 1–2
    "Shattered God": Math.floor(Math.random() * 4) + 1 // 1–4
  };
} else {
  // Use exact counts from frontmatter
  tierCounts = {
    "True God": pantheonSettings.amtTrueGod,
    "Lesser God": pantheonSettings.amtLesserGod,
    "Demigod": pantheonSettings.amtDemigod,
    "Angel": pantheonSettings.amtAngel,
    "Saint": pantheonSettings.amtSaint,
    "Champion": pantheonSettings.amtChampion,
    "False God": pantheonSettings.amtFalseGod,
    "Shattered God": pantheonSettings.amtShatteredGod
  };
}

// Mapping of tier to callout tag and color
const tierCalloutTypes = {
  "True God": "god-true",
  "Lesser God": "god-lesser",
  "Demigod": "god-demigod",
  "Angel": "god-angel",
  "Saint": "god-saint",
  "Champion": "god-champion",
  "False God": "god-false",
  "Shattered God": "god-shattered"
};

const tierColorHex = {
  "True God": "#f00",
  "Lesser God": "#f90",
  "Demigod": "#ff0",
  "Angel": "#0f0",
  "Saint": "#0ff",
  "Champion": "#00f",
  "False God": "#90f",
  "Shattered God": "#f0f"
};

// Generate array of god objects for all tiers
const gods = Object.entries(tierCounts).flatMap(([tier, count]) =>
  Array.from({ length: count }, () => generateGod(tier))
);

/*
  ==================================
  Section: Relationship Assignment
  ==================================
*/

// Possible relationship pairs (e.g., Parent of / Child of, etc.)
const RELATIONSHIP_PAIRS = [
  ...Array(4).fill(["Parent of", "Child of"]),
  ...Array(3).fill(["Sibling of", "Sibling of"]),
  ["Friend of", "Friend of"],
  ["Mentor of", "Student of"],
  ["Lover of", "Lover of"],
  ["Rival to", "Rival to"],
  ["Sworn Enemy of", "Sworn Enemy of"]
];

// Initialize empty relationships for each god
gods.forEach((g) => (g.relationships = []));

// Assign random relationships between gods
const relationshipCount = Math.floor(gods.length / 2);
const maxPairs = Math.floor(gods.length * 1.5);
let attempts = 0;
while (attempts < maxPairs) {
  const i = Math.floor(Math.random() * gods.length);
  let j = Math.floor(Math.random() * gods.length);

  if (
    i !== j &&
    !gods[i].relationships.some((r) => r.includes(`[[${gods[j].name}]]`))
  ) {
    const [typeA, typeB] = pick(RELATIONSHIP_PAIRS);
    gods[i].relationships.push(`${typeA} [[${gods[j].name}]]`);
    gods[j].relationships.push(`${typeB} [[${gods[i].name}]]`);
    attempts++;
  }
}

/*
  ======================
  Section: Rendering
  ======================
*/

// Format a single god into a callout string
const formatGod = (god) => `
> [!recite|no-t]
> # [[${god.name}]] *(${god.tier})*
> ## *${god.epithet}*
> | Attribute | Value |
> |---------|------|
> | Pantheon   | ${pantheon} |
> | Domain     | ${god.domain} |
> | Alignment  | ${god.alignment} |
>
> | Attribute | Value |
> |---------|-------|
> | Aspects   | ${god.aspects.join(", ")} |
> | Symbols   | ${god.mainSymbol}, ${god.subSymbols.join(", ")} |
> | Colors    | ${god.colors.join(", ")} |
>
> | Attribute | Value |
> |------|------------|
> | Patrons    | ${god.patrons.join(", ")} |
> | Holy Day   | ${god.holyDay || ""} |
> ${god.realm ? `> \n> **Realm**: _${god.realm}_` : ""}
> ${god.ruinSite ? `> \n> **Ruin Site**: _${god.ruinSite}_` : ""}
> ${god.serves ? `> \n> **Serves**: _${god.serves}_` : ""}
> ${god.createdBy ? `> \n> **Created By**: _${god.createdBy}_` : ""}
> ${god.ambition ? `> \n> **Ambition**: _${god.ambition}_` : ""}
> ${god.mission ? `> \n> **Mission**: _${god.mission}_` : ""}
> ${god.martyrdom ? `> \n> **Martyrdom**: _${god.martyrdom}_` : ""}
> ${god.relationships.length ? `> \n> **Relationships**: ${god.relationships.join(", ")}` : ""}
>
> ### Tenet
> _${god.tenet}_
>
> ### Myth
> _${god.myth}_
`;

// Build summary table header
let summaryTable = 
  "| Deity | Tier | Domain | Alignment | Aspects | Patrons | Symbols | Realm | Serves | Created By | Martyrdom | Ruin Site | Relationships |\n" +
  "|-------|------|--------|-----------|---------|---------|---------|-------|--------|-------------|----------|-----------|---------------|\n";

// Populate summary table rows
for (const god of gods) {
  const symbolDisplay = [god.mainSymbol, ...(god.subSymbols || [])]
    .filter(Boolean)
    .join(", ") || "–";
  summaryTable += `| [[${god.name}]] | ${god.tier} | ${god.domain} | ${god.alignment} | ${god.aspects?.join(", ") || "–"} | ${god.patrons?.join(", ") || "–"} | ${symbolDisplay} | ${god.realm || "–"} | ${god.serves || "–"} | ${god.createdBy || "–"} | ${god.martyrdom || "–"} | ${god.ruinSite || "–"} | ${(god.relationships && god.relationships.length > 0 ? god.relationships.join("; ") : "–")} |\n`;
}

// Create Mermaid graph showing parent-child and sibling relationships
let mermaidDiagram = "```mermaid\ngraph TD\n";
const childToParents = new Map();

// Step 1: Build child → [parents] map
for (const god of gods) {
  for (const rel of god.relationships) {
    const match = rel.match(/Parent of \[\[(.*?)\]\]/);
    if (!match) continue;
    const child = match[1];
    if (!childToParents.has(child)) childToParents.set(child, new Set());
    childToParents.get(child).add(god.name);
  }
}

// Step 2: Inherit missing parent sets from siblings
for (const god of gods) {
  if (childToParents.has(god.name)) continue;
  const siblingMatches = god.relationships
    .map((rel) => rel.match(/Sibling of \[\[(.*?)\]\]/))
    .filter(Boolean)
    .map((m) => m[1]);
  for (const sibling of siblingMatches) {
    if (childToParents.has(sibling)) {
      childToParents.set(god.name, new Set(childToParents.get(sibling)));
      break;
    }
  }
}

// Step 3: Group children by identical parent sets
const parentSetKeyToChildren = new Map();
for (const [child, parentSet] of childToParents.entries()) {
  const key = [...parentSet].sort().join("|");
  if (!parentSetKeyToChildren.has(key)) {
    parentSetKeyToChildren.set(key, { parents: [...parentSet].sort(), children: new Set() });
  }
  parentSetKeyToChildren.get(key).children.add(child);
}

// Step 4: For each unique parent set, connect to synthetic node
let synthIndex = 0;
for (const { parents, children } of parentSetKeyToChildren.values()) {
  const synthId = `synth${synthIndex++}`;
  for (const parent of parents) {
    mermaidDiagram += `${parent} --> ${synthId}( )\n`;
  }
  for (const child of children) {
    mermaidDiagram += `${synthId} --> ${child}\n`;
  }
}

// Step 5: Add standalone gods to the diagram
const allLinked = new Set();
for (const child of childToParents.keys()) {
  allLinked.add(child);
}
for (const parentSet of childToParents.values()) {
  for (const parent of parentSet) {
    allLinked.add(parent);
  }
}
for (const god of gods) {
  const siblingMatches = god.relationships
    .map((rel) => rel.match(/Sibling of \[\[(.*?)\]\]/))
    .filter(Boolean)
    .map((m) => m[1]);
  for (const sibling of siblingMatches) {
    allLinked.add(god.name);
    allLinked.add(sibling);
  }
}
for (const god of gods) {
  if (!allLinked.has(god.name)) {
    mermaidDiagram += `${god.name}\n`;
  }
}

// Sanitize names for Mermaid and assign classes
const sanitizeName = (name) => name.replace(/[^a-zA-Z0-9_]/g, "_");
for (const god of gods) {
  const className = `god-${god.tier.toLowerCase().replace(/ /g, "-")}`;
  mermaidDiagram += `class ${sanitizeName(god.name)} ${className}\n`;
}
for (const [tier, color] of Object.entries(tierColorHex)) {
  const className = `god-${tier.toLowerCase().replace(/ /g, "-")}`;
  mermaidDiagram += `classDef ${className} stroke:${color};\n`;
}
for (let i = 0; i < synthIndex; i++) {
  mermaidDiagram += `class synth${i} synthNode\n`;
}
mermaidDiagram += `classDef synthNode stroke:#999,fill:#fff,stroke-width:1px;\n`;
mermaidDiagram += "```";

/*
  =================================
  Section: Output to Template (tR)
  =================================
*/

// Output Mermaid diagram (full-width, no callout)
tR += `\n\n\`\`\`mermaid\n${mermaidDiagram.replace(/^```mermaid\n|\n```$/g, '')}\n\`\`\`\n\n`;

// Output collapsible summary table
tR += `\n\n> [!summary|bg-c-plain]+ Summary Table\n` +
  summaryTable.split('\n').map((line) => `> ${line}`).join('\n');

// Output callouts for each god
for (let i = 0; i < gods.length; i++) {
  const god = gods[i];
  const calloutType = tierCalloutTypes[god.tier] || "god-default";
  const header = `> [!${calloutType}]- **[[${god.name}]]** (${god.tier})`;
  const body = formatGod(god).split('\n').map((line) => `> ${line}`).join('\n');
  tR += `\n\n${header}\n${body}`;
}

/*
  ================================
  Section: File Creation/Export
  ================================
*/

// Ensure a folder exists (create if missing)
const ensureFolderExists = async (path) => {
  const existing = app.vault.getAbstractFileByPath(path);
  if (!existing) {
    await app.vault.createFolder(path);
  }
};

// Rename this Pantheon file to include a unique index if duplicates exist
const deityFolder = app.vault.getFolderByPath("2-World/Cosmology/Deities");
const baseName = `${pantheon} Pantheon`;
let count = 1;
for (const child of deityFolder.children) {
  if (child.name.endsWith(".md") && child.name.startsWith(baseName)) {
    count++;
  }
}
const finalName = count === 1 ? baseName : `${baseName} ${count - 1}`;
await tp.file.rename(finalName);

// Create a note for each generated god under the appropriate tier folder
const tierNumberMap = {
  "True God": "1",
  "Lesser God": "2",
  "Demigod": "3",
  "Angel": "4",
  "Saint": "5",
  "Champion": "6",
  "False God": "7",
  "Shattered God": "8"
};
for (const god of gods) {
  const folderPath = `2-World/Cosmology/Deities/${pantheon}/${tierNumberMap[god.tier]}. ${god.tier}s`;
  const fileName = `${god.name}.md`;
  const yamlBlock = `---
name: ${god.name}
epithet: "${god.epithet}"
tier: ${god.tier}
pantheon: ${pantheon}
domain: ${god.domain}
alignment: ${god.alignment}
aspects: [${god.aspects.map(a => `"${a}"`).join(", ")}]
symbols:
  main: "${god.mainSymbol}"
  sub: [${god.subSymbols.map(s => `"${s}"`).join(", ")}]
colors: [${god.colors.map(c => `"${c}"`).join(", ")}]
patrons: [${god.patrons.map(p => `"${p}"`).join(", ")}]
holy_day: ${god.holyDay ? `"${god.holyDay}"` : ""}
tenet: "${god.tenet}"
myth: "${god.myth}"
realm: ${god.realm ? `"${god.realm}"` : ""}
ruin_site: ${god.ruinSite ? `"${god.ruinSite}"` : ""}
serves: ${god.serves ? `"${god.serves}"` : ""}
created_by: ${god.createdBy ? `"${god.createdBy}"` : ""}
ambition: ${god.ambition ? `"${god.ambition}"` : ""}
martyrdom: ${god.martyrdom ? `"${god.martyrdom}"` : ""}
mission: ${god.mission ? `"${god.mission}"` : ""}
relationships: [${god.relationships.map(r => `"${r}"`).join(", ")}]
is_goddess: ${god.isGoddess}
---`;

const fileContent = `${yamlBlock}\n\n${formatGod(god)}`;


  await ensureFolderExists(folderPath);
  await app.vault.create(`${folderPath}/${fileName}`, fileContent);
};
%>
