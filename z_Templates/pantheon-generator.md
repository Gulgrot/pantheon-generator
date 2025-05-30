<%*
// Load Data
const loadValues = async (filepath) => {
  const file = app.vault.getAbstractFileByPath(filepath);
  if (!file) return [];
  const content = await app.vault.read(file);
  return content.split('\n').map(line => line.trim()).filter(line => line.length > 0);
};

const pick = (arr) => arr[Math.floor(Math.random() * arr.length)];
const pickMultiple = (arr, count) => {
  const result = [];
  const available = [...arr];
  for (let i = 0; i < count && available.length > 0; i++) {
    const idx = Math.floor(Math.random() * available.length);
    result.push(available.splice(idx, 1)[0]);
  }
  return result;
};

const pluralize = (word) => {
  if (word.endsWith("s")) return word;
  if (word.endsWith("y")) return word.slice(0, -1) + "ies";
  return word + "s";
};

const conjugateVerb = (verb, form) => {
  if (form === "s") {
    if (verb.endsWith("y") && !"aeiou".includes(verb[verb.length - 2])) {
      return verb.slice(0, -1) + "ies";
    } else if (["s", "x", "z"].includes(verb.slice(-1)) || /ch$|sh$/.test(verb)) {
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

const capitalize = (str) => str.charAt(0).toUpperCase() + str.slice(1);
const capitalizeTitle = str =>
  str.split(" ").map(word =>
    word.includes("-")
      ? word.split("-").map(w => capitalize(w)).join("-")
      : capitalize(word)
  ).join(" ");

const [
  adjectives, adverbs, nouns, verbs, 
  alignments, ambitions, aspects, colors, createdBy, domains, epithets, hierarchy, holyDays, loyalty, martydom, missions, myths, names, pantheons, patrons, realms, relationships, ruins, symbols, tenets
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
  loadValues("z_Generators/Pantheon Generator/data/martydom.md"),
  loadValues("z_Generators/Pantheon Generator/data/missions.md"),
  loadValues("z_Generators/Pantheon Generator/data/myths.md"),
  loadValues("z_Generators/Pantheon Generator/data/names.md"),
  loadValues("z_Generators/Pantheon Generator/data/pantheons.md"),
  loadValues("z_Generators/Pantheon Generator/data/patrons.md"),
  loadValues("z_Generators/Pantheon Generator/data/realms.md"),
  loadValues("z_Generators/Pantheon Generator/data/relationships.md"),
  loadValues("z_Generators/Pantheon Generator/data/ruins.md"),
  loadValues("z_Generators/Pantheon Generator/data/symbols.md"),
  loadValues("z_Generators/Pantheon Generator/data/tenets.md")
]);

const DEFAULT_WORD_BANK = {
  nounList: nouns,
  adjList: adjectives,
  advList: adverbs,
  verbList: verbs,
  colorList: colors
};

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

const enrichByTier = (god) => {
  switch (god.tier) {
    case "True God":
      god.realm = capitalizeTitle(renderTemplate(pick(realms)));
      god.aspects = pickMultiple(aspects, 3);
      god.subSymbols = pickMultiple(nouns, 2).map(noun => capitalize(pluralize(noun)));
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
      god.martyrdom = pick(martydom);
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
      god.martyrdom = pick(martydom);
      god.ruinSite = capitalizeTitle(renderTemplate(pick(ruins)));
      break;
  }
};

const pantheon = pick(pantheons);
const generateGod = (tier) => {
  const god = {
    name: pick(names),
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
    subSymbols: pickMultiple(nouns, Math.floor(Math.random() * 2) + 1).map(noun => capitalize(pluralize(noun))),
  };
  enrichByTier(god);
  return god;
};

const tierCounts = {
  "True God": Math.floor(Math.random() * 5) + 8,       // 8–12
  "Lesser God": Math.floor(Math.random() * 3) + 4,     // 4–6
  "Demigod": Math.floor(Math.random() * 2) + 1,        // 1–2
  "Angel": Math.floor(Math.random() * 3) + 2,          // 2–4
  "Saint": Math.floor(Math.random() * 3) + 2,          // 2–4
  "Champion": Math.floor(Math.random() * 2) + 1,       // 1–2
  "False God": Math.floor(Math.random() * 2) + 1,      // 1–2
  "Shattered God": Math.floor(Math.random() * 4) + 1   // 1–4
};

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

const tierLucideIcons = {
  "True God": "sparkles",
  "Lesser God": "sparkle",
  "Demigod": "flame",
  "Angel": "feather",
  "Saint": "heart",
  "Champion": "trophy",
  "False God": "shield-x",
  "Shattered God": "zap-off"
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


const gods = Object.entries(tierCounts).flatMap(([tier, count]) =>
  Array.from({ length: count }, () => generateGod(tier))
);

const RELATIONSHIP_PAIRS = [
  ...Array(3).fill(["Parent of", "Child of"]),
  ...Array(3).fill(["Sibling of", "Sibling of"]),
  ...Array(2).fill(["Friend of", "Friend of"]),
  ["Mentor of", "Student of"],
  ["Lover of", "Lover of"],
  ["Rival to", "Rival to"],
  ["Sworn Enemy of", "Sworn Enemy of"]
];

// Add blank relationship field
gods.forEach(g => g.relationships = []);

// Create a number of pairings
const relationshipCount = Math.floor(gods.length / 2);
const pairedIndexes = [];

const maxPairs = Math.floor(gods.length * 1.5); // adjust multiplier for more/fewer relationships
let attempts = 0;

while (attempts < maxPairs) {
  const i = Math.floor(Math.random() * gods.length);
  let j = Math.floor(Math.random() * gods.length);

  // Ensure not the same god and relationship doesn't already exist
  if (i !== j && !gods[i].relationships.some(r => r.includes(`[[${gods[j].name}]]`))) {
    const [typeA, typeB] = pick(RELATIONSHIP_PAIRS);
    gods[i].relationships.push(`${typeA} [[${gods[j].name}]]`);
    gods[j].relationships.push(`${typeB} [[${gods[i].name}]]`);
    attempts++;
  }
}

const calloutTags = ["left", "center", "right"];
const formatGod = (god, position) => `
> [!recite|no-t]
> # [[${god.name}]] *(${god.tier})*
> ## *${god.epithet}*
> | Attribute | Value |
> |---------|------|
> | Pantheon | ${pantheon} |
> | Domain | ${god.domain} |
> | Alignment | ${god.alignment} |
>
> | Attribute | Value |
> |---------|-------|
> | Aspects | ${god.aspects.join(", ")} |
> | Symbols | ${god.mainSymbol}, ${god.subSymbols.join(", ")} |
> | Colors | ${god.colors.join(", ")} |
>
> | Attribute | Value |
> |------|------------|
> | Patrons | ${god.patrons.join(", ")} |
> | Holy Day | ${god.holyDay || ""} |
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

let summaryTable = "| Deity | Tier | Domain | Alignment | Aspects | Patrons | Symbols | Realm | Serves | Created By | Martydom | Ruin Site | Relationships |\n";
summaryTable += "|-------|------|--------|-----------|---------|---------|---------|-------|--------|-------------|-------------|-------------|----------------|\n";
for (const g of gods) {
  const symbolDisplay = [g.mainSymbol, ...(g.subSymbols || [])].filter(Boolean).join(", ") || "–";
  summaryTable += `| [[${g.name}]] | ${g.tier} | ${g.domain} | ${g.alignment} | ${g.aspects?.join(", ") || "–"} | ${g.patrons?.join(", ") || "–"} | ${symbolDisplay} | ${g.realm || "–"} | ${g.serves || "–"} | ${g.createdBy || "–"} | ${g.martyrdom || "–"} | ${g.ruinSite || "–"} | ${(g.relationships && g.relationships.length > 0 ? g.relationships.join("; ") : "–")} |\n`;
}

let mermaidDiagram = "```mermaid\ngraph TD\n";

const childToParents = new Map();

// Step 1: Build child → [parents] map
for (const g of gods) {
  const from = g.name;
  for (const rel of g.relationships) {
    const match = rel.match(/Parent of \[\[(.*?)\]\]/);
    if (!match) continue;
    const child = match[1];
    if (!childToParents.has(child)) childToParents.set(child, new Set());
    childToParents.get(child).add(from);
  }
}

// Step 2: Inherit missing parent sets from siblings
for (const g of gods) {
  const name = g.name;
  if (childToParents.has(name)) continue;

  const siblingMatches = g.relationships
    .map(rel => rel.match(/Sibling of \[\[(.*?)\]\]/))
    .filter(Boolean)
    .map(m => m[1]);

  for (const sibling of siblingMatches) {
    if (childToParents.has(sibling)) {
      childToParents.set(name, new Set(childToParents.get(sibling)));
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

// Collect from parent-child links
for (const child of childToParents.keys()) {
  allLinked.add(child);
}
for (const parentSet of childToParents.values()) {
  for (const parent of parentSet) {
    allLinked.add(parent);
  }
}

// Collect from sibling links
for (const g of gods) {
  const name = g.name;
  const siblingMatches = g.relationships
    .map(rel => rel.match(/Sibling of \[\[(.*?)\]\]/))
    .filter(Boolean)
    .map(m => m[1]);

  for (const sibling of siblingMatches) {
    allLinked.add(name);
    allLinked.add(sibling);
  }
}

// Render each unlinked god as a floating node
for (const g of gods) {
  if (!allLinked.has(g.name)) {
    mermaidDiagram += `${g.name}\n`;
  }
}

// Mermaid-safe names (no spaces, special characters)
const sanitizeName = name => name.replace(/[^a-zA-Z0-9_]/g, "_");

// Add class assignments for each god
for (const g of gods) {
  const className = `god-${g.tier.toLowerCase().replace(/ /g, "-")}`;
  mermaidDiagram += `class ${sanitizeName(g.name)} ${className}\n`;
}

// Add classDefs with stroke colors
for (const [tier, color] of Object.entries(tierColorHex)) {
  const className = `god-${tier.toLowerCase().replace(/ /g, "-")}`;
  mermaidDiagram += `classDef ${className} stroke:${color};\n`;
}
for (let i = 0; i < synthIndex; i++) {
  mermaidDiagram += `class synth${i} synthNode
`;
}
mermaidDiagram += `classDef synthNode stroke:#999,fill:#fff,stroke-width:1px;
`;

mermaidDiagram += "```";


// Output plain Mermaid diagram (full-width, no callout)
tR += `\n\n\`\`\`mermaid\n${mermaidDiagram.replace(/^```mermaid\n|\n```$/g, '')}\n\`\`\`\n\n`;

// Output collapsible summary table
tR += `\n\n> [!summary|bg-c-plain]+ Summary Table\n` +
      summaryTable.split('\n').map(line => `> ${line}`).join('\n');

tR += "\n\n" + gods.map((g, i) => {
  const calloutType = tierCalloutTypes[g.tier] || "god-default";
  const header = `> [!${calloutType}]- **[[${g.name}]]** (${g.tier})`;
  const body = formatGod(g, calloutTags[i % calloutTags.length])
    .split('\n')
    .map(line => `> ${line}`)
    .join('\n');
  return `${header}\n${body}`;
}).join("\n\n");

%>
