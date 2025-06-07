```dataviewjs
// 1) root container
const container = document.createElement("div");
container.className = "pantheon-dashboard";
dv.container.appendChild(container);

// 2) SEARCH + CLEAR BAR
const topBar = document.createElement("div");
topBar.className = "search-container";
container.appendChild(topBar);

// 2a) Name search
const searchInput = document.createElement("input");
searchInput.type = "text";
searchInput.id = "god-search";
searchInput.placeholder = "🔍 Search by name…";
searchInput.classList.add("search-input");
topBar.appendChild(searchInput);

// 2b) Aspect search
const aspectInput = document.createElement("input");
aspectInput.type = "text";
aspectInput.id = "aspect-search";
aspectInput.placeholder = "🔍 Search by aspect…";
aspectInput.classList.add("search-input");
topBar.appendChild(aspectInput);

// 2c) Clear button
const clearBtn = document.createElement("button");
clearBtn.textContent = "Clear All Filters";
clearBtn.className = "clear-filters-btn";
topBar.appendChild(clearBtn);

// 3) FILTER CHECKBOXES
const filterContainer = document.createElement("div");
filterContainer.className = "filter-grid";
container.appendChild(filterContainer);

const filters = {
  alignment: [
    "Lawful Good","Neutral Good","Chaotic Good",
    "Lawful Neutral","True Neutral","Chaotic Neutral",
    "Lawful Evil","Neutral Evil","Chaotic Evil"
  ],
  domain: [
    "Arcana","Death","Forge","Grave","Knowledge",
    "Life","Light","Nature","Order","Peace",
    "Tempest","Trickery","Twilight","War"
  ].sort(),
  tier: [
    "True God","Lesser God","Demigod",
    "Angel","Saint","Champion",
    "False God","Shattered God"
  ],
  pantheon: [
    "Human","Dwarf","Tiefling","Dragonborn",
    "Goblin","Orc","Halfling","Gnome","Aasimar",
    "Drow","Goliath","Wood Elf","High Elf"
  ]
};

const checkboxes = {};
for (const [key, values] of Object.entries(filters)) {
  const section = document.createElement("div");
  section.className = `filter-group filter-${key}`;
  const heading = document.createElement("strong");
  heading.textContent = key.toUpperCase();
  section.appendChild(heading);

  const optionsContainer = document.createElement("div");
  optionsContainer.className = "filter-options";
  checkboxes[key] = [];

  for (const value of values) {
    const id = `${key}-${value.replace(/\s+/g, "-")}`;
    const label = document.createElement("label");
    label.setAttribute("for", id);

    const checkbox = document.createElement("input");
    checkbox.type = "checkbox";
    checkbox.id = id;
    checkbox.value = value;
    checkbox.dataset.filterKey = key;
    checkboxes[key].push(checkbox);

    label.appendChild(checkbox);
    label.append(` ${value}`);
    optionsContainer.appendChild(label);
  }

  section.appendChild(optionsContainer);
  filterContainer.appendChild(section);
}

// 4) RESULTS SLOT
const resultDiv = document.createElement("div");
resultDiv.className = "results";
container.appendChild(resultDiv);

// 5) MAIN RENDER FUNCTION
function updateResults() {
  const searchTerm = searchInput.value.trim().toLowerCase();
  const aspectTerm = aspectInput.value.trim().toLowerCase();

  // which boxes are checked?
  const selected = {};
  for (const key of Object.keys(filters)) {
    selected[key] = checkboxes[key]
      .filter(cb => cb.checked)
      .map(cb => cb.value);
  }

  // fetch & filter pages
  const notes = dv
    .pages('"2-World/Cosmology/Deities"')
    .where(p => p.tier)  // drop non-deity files
    .where(p =>
      (!selected.alignment.length || selected.alignment.includes(p.alignment)) &&
      (!selected.domain.length    || selected.domain.includes(p.domain))    &&
      (!selected.tier.length      || selected.tier.includes(p.tier))       &&
      (!selected.pantheon.length  || selected.pantheon.includes(p.pantheon))
    )
    .where(p =>
      (!searchTerm || (p.name && p.name.toLowerCase().includes(searchTerm))) &&
      (!aspectTerm || (
        p.aspects &&
        (Array.isArray(p.aspects)
          ? p.aspects.some(a => a.toLowerCase().includes(aspectTerm))
          : p.aspects.toLowerCase().includes(aspectTerm)
        )
      ))
    );

  // clear old table
  resultDiv.innerHTML = "";

  // build new HTML table
  const table = document.createElement("table");
  table.className = "pantheon-table";
  const headers = ["Name","Pantheon","Tier","Alignment","Domain","Aspects","Realm"];
  const thead = table.createTHead();
  const hdrRow = thead.insertRow();
  for (const h of headers) {
    const th = document.createElement("th");
    th.textContent = h;
    hdrRow.appendChild(th);
  }
  const tbody = table.createTBody();
  for (const god of notes) {
    const tr = tbody.insertRow();

    // Name with Obsidian preview/open
    const nameTd = tr.insertCell();
    if (god.file?.path) {
      const a = document.createElement("a");
      a.setAttribute("href", god.file.path);
      a.classList.add("internal-link");
      a.textContent = god.name;
      nameTd.appendChild(a);
    } else {
      nameTd.textContent = god.name;
    }

    // other columns
    const rest = [
      god.pantheon,
      god.tier,
      god.alignment,
      god.domain,
      Array.isArray(god.aspects) ? god.aspects.join(", ") : (god.aspects||""),
      god.realm || ""
    ];
    for (const val of rest) {
      const td = tr.insertCell();
      td.textContent = val;
    }
  }

  resultDiv.appendChild(table);
}

// 6) EVENT HOOKUPS
searchInput.addEventListener("input", updateResults);
aspectInput.addEventListener("input", updateResults);
Object.values(checkboxes).flat()
  .forEach(cb => cb.addEventListener("change", updateResults));

clearBtn.addEventListener("click", () => {
  // uncheck all
  Object.values(checkboxes).flat()
    .forEach(cb => cb.checked = false);
  // clear searches
  searchInput.value = "";
  aspectInput.value = "";
  updateResults();
});

// 7) initial draw
updateResults();
```
