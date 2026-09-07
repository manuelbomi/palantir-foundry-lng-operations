# Palantir Foundry for LNG Operations — Building an LNG Cargo Command Console with Workshop

**How the same Foundry Workshop mechanics an FDE learns in training become an LNG terminal's cargo command console — with real, hands-on screenshots and an explicit "how this translates" layer for the energy industry**

[![Platform](https://img.shields.io/badge/Platform-Palantir%20Foundry-1a1a2e?style=flat-square)](https://www.palantir.com/platforms/foundry/)
[![Tool](https://img.shields.io/badge/Tool-Workshop-0f6fde?style=flat-square)]()
[![Industry](https://img.shields.io/badge/Industry-LNG%20%2F%20Energy-f57f17?style=flat-square)]()
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

> This is a companion build to the 4-part Palantir Foundry FDE series — [Data Pipelines](https://github.com/manuelbomi/palantir-foundry-data-pipelines) → [Ontology](https://github.com/manuelbomi/palantir-foundry-ontology) → [Workshop Apps](https://github.com/manuelbomi/palantir-foundry-workshop-apps) → [The FDE Playbook](https://github.com/manuelbomi/palantir-foundry-fde-playbook). Where that series builds a generic order-fulfillment app, this repo takes the exact same Workshop mechanics — captured first-hand, not from a training document — and reframes them for a different vertical: **LNG (Liquefied Natural Gas) shipping and terminal operations.**

---

## Table of Contents

1. [A Note on What These Screenshots Actually Show](#a-note-on-what-these-screenshots-actually-show)
2. [The LNG Operations Problem](#the-lng-operations-problem)
3. [Why LNG Is an Unusually Good Fit for Foundry](#why-lng-is-an-unusually-good-fit-for-foundry)
4. [What We're Building: The LNG Cargo Ops Console](#what-were-building-the-lng-cargo-ops-console)
5. [Field Mapping: Generic Order → LNG Cargo](#field-mapping-generic-order--lng-cargo)
6. [Walkthrough, With LNG Translation Notes](#walkthrough-with-lng-translation-notes)
7. [Roadmap: What a Production LNG Console Adds Next](#roadmap-what-a-production-lng-console-adds-next)
8. [Design Decisions an FDE Has to Defend](#design-decisions-an-fde-has-to-defend)
9. [Beyond LNG: The Same Console Pattern in Other Energy Verticals](#beyond-lng-the-same-console-pattern-in-other-energy-verticals)
10. [Why Enterprises in Energy Choose Palantir](#why-enterprises-in-energy-choose-palantir)
11. [Repo Contents](#repo-contents)
12. [Related Repositories](#related-repositories)

---

## A Note on What These Screenshots Actually Show

In the interest of accuracy: the 26 screenshots in this repo are real, first-hand captures from a live Foundry training tenant, taken while building the generic "Orders Inbox" module from Palantir's official Foundry tutorial — the same module modeled around retail order fulfillment used in the [Workshop Apps](https://github.com/manuelbomi/palantir-foundry-workshop-apps) repo. They literally show an `Order` object, an `Item Name` property, and a module called `[username] Orders Inbox`.

Nothing about *that* is LNG-specific — and that's the point of this repo. **The mechanics of building a Workshop module — creating it, naming it, adding an object table, wiring a filter — are 100% industry-agnostic.** What changes between a retail order console and an LNG cargo console isn't the button you click; it's *what object you point those widgets at* and *what you name things once you get there*. Every step below keeps its literal caption (what's actually on screen) paired with an explicit **LNG Translation** note (what an FDE would do differently, or name differently, if the object behind the same widget were an LNG cargo instead of a retail order). Nothing here claims the screenshots show LNG data — they don't. What they show, faithfully, is the reusable skill.

## The LNG Operations Problem

Picture this instead of a paper-goods merger:

> You're deployed to an LNG exporter running a two-train liquefaction terminal. Commercial, marine operations, and the terminal duty office each track the same thing — **cargoes moving through the terminal** — in three different places: a chartering spreadsheet tracking laycans and offtakers, the marine operations system (MOS) tracking berth schedules and vessel ETAs, and a duty officer's handover log tracking boil-off gas readings and loading status. None of them agree on which cargo is at risk of missing its laycan window, which is the single most expensive kind of mistake in LNG shipping: miss a laycan, and the charter party's demurrage clause turns a schedule slip into a six- or seven-figure penalty.

This is structurally the *same* problem the rest of this series solves for order fulfillment — multiple systems, one real-world process, no shared source of truth — just with cryogenic tankers and nine-figure cargoes instead of staplers and monitors.

## Why LNG Is an Unusually Good Fit for Foundry

A few properties of LNG operations make this a genuinely strong match for Foundry's model, not just a plausible-sounding one:

- **The assets are enormous and instrumented.** A liquefaction train, a storage tank, and an LNG carrier are each covered in sensors — temperature, pressure, boil-off rate, flow. Palantir's [Foundry for Energy](https://www.palantir.com/offerings/energy/) and [Vertex digital twin for oil & gas](https://www.palantir.com/vertex-for-energy/) offerings exist specifically to turn that sensor firehose into a governed, queryable model, rather than another dashboard nobody trusts.
- **Palantir already has deep, long-running traction in this exact space.** Palantir and **bp** have partnered since 2014, with Foundry powering bp's reliability program and a model-based digital twin integrating dynamic asset models with real-time data from **over two million sensors** into one operating picture ([Palantir Blog](https://blog.palantir.com/how-palantir-foundry-powers-bps-digital-transformation-in-reliability-4c644e36b6fc)) — bp has since committed to five more years of Foundry, extending it into wind, solar, and EV charging.
- **The stakes for getting "one source of truth" wrong are unusually high.** A demurrage dispute, a missed laycan, or a boil-off gas incident isn't a bad quarterly report — it's a contractual penalty or a safety event. That's exactly the class of problem an Ontology-backed operational console (not a static BI dashboard) is built to prevent, because it's built to be *acted on*, not just read.
- **The org chart mirrors the "two systems" pattern perfectly.** Chartering/commercial, marine operations, and terminal safety functionally run the same process — moving a cargo from nomination to discharge — through three different systems of record. That's precisely the integration problem Foundry's pipeline → Ontology → app chain is designed to solve.

## What We're Building: The LNG Cargo Ops Console

Reusing the exact widget sequence captured in the screenshots — module creation, object table, sort/rename, layout, filter — but pointed at an LNG cargo model instead of a retail order model:

```
Cargo (Ontology Object, backed by chartering + MOS + terminal data)
      │
      ▼
LNG Cargo Ops Console (Workshop module)
   ├─ Cargo Table         — every active cargo, sortable by Vessel Name
   ├─ Filter Panel         — by Berth, Charterer, Cargo Status
   └─ (roadmap) Charts, laycan-risk view, Actions — see below
```

## Field Mapping: Generic Order → LNG Cargo

This table is the actual translation an FDE performs — the same widget, pointed at a different property:

| Training tutorial field | LNG Cargo Ops Console field | What it represents |
|---|---|---|
| `Order Id` | `Cargo Id` | Unique identifier for one LNG cargo/shipment |
| `Item Name` (Object title) | `Vessel Name` | The LNG carrier performing the voyage — the name a duty officer actually says out loud |
| `Status` | `Cargo Status` | `Nominated → Loading → Laden (In Transit) → Discharging → Completed` |
| `Assignee` | `Marine Operator` / `Terminal Duty Officer` | The person accountable for this cargo right now |
| `Customer Name` | `Offtaker / Charterer` | The buyer or chartering counterparty for this cargo |
| `Days Until Due` | `Days Until Laycan Expiry` | The single number that determines demurrage exposure |
| `Order Due Date` | `Laycan End Date` | The last day the vessel can complete loading under the charter party |
| `Quantity` | `Cargo Volume (m³ LNG)` | Volume of LNG being loaded/discharged |
| `Unit Price` | `Contract Price ($/MMBtu)` | Commercial value of the cargo |

## Walkthrough, With LNG Translation Notes

### Phase 1 — Stand up the module

**What's shown:** navigating to the backing Object Type from Ontology Manager, then opening its Workshop tab and creating the first module.

![Open your object in Ontology Manager](images/01-open-your-username-object-in-ontology-manager.png)
![Click Overview, Dependencies, Workshop](images/02-click-on-overview-dependent-workshop.png)
![Click Create your first module](images/03-click-on-create-your-first.png)

> **LNG Translation:** the Object opened here would be `Cargo`, backed by a pipeline that's already joined chartering, MOS, and terminal duty-log data — the LNG equivalent of `all_orders` from [Part 1](https://github.com/manuelbomi/palantir-foundry-data-pipelines) of the main series.

**What's shown:** naming the module `[username] Orders Inbox`, picking a cube icon, and setting its color to Blue 4.

![Rename module](images/04-rename-to-username-orders-inbox.png)
![After renaming](images/05-after-renaming.png)
![Pick a cube icon](images/06-rename-and-under-icons-search-for-and-select-cube.png)
![Set color to Blue 4](images/07-set-colour-to-blue-4.png)

> **LNG Translation:** the module gets named **"LNG Cargo Ops Console"**, with a ship/tanker icon in place of the cube — and Blue 4 stays exactly as-is. It's not a coincidence that Foundry's default palette already reads as "maritime/energy" without changing a thing; small details like this are part of why a console feels purpose-built rather than generic on day one.

### Phase 2 — Build the Cargo Table

**What's shown:** adding an Object Table widget, wiring it to a new object set variable, and selecting the backing object set.

![Click Add Widget](images/08-click-add-widget.png)
![Add Object Table widget](images/09-add-object-table-widget.png)
![Object Set Variable](images/10-object-set-variable.png)
![Select New Object Set Variable](images/11-select-new-object-set-variable.png)
![Select Order object set](images/12-select-order-object-set-for-your-order.png)
![Select Order object set](images/12b-select-order-object-set.png)
![Select Order object set, confirm](images/13-select-order-object-set.png)
![Rename the variable to keep track](images/14-rename-to-be-able-to-keep-track.png)

> **LNG Translation:** the object set selected here is `Cargo`, and the variable is renamed **"Cargo Object Set"** rather than left as a generic `var1` — a habit worth keeping regardless of industry, since an undocumented variable named `var1` is the first thing that makes a module unmaintainable for the next engineer who opens it.

**What's shown:** adding every property to the table, choosing a sort property, and renaming the default `Title` column.

![Select Add All Properties](images/15-select-add-all-properties.png)
![Select a property to sort by](images/16-select-a-property-to-sort-by.png)
![Rename the title property](images/17-rename-by-clicking-to-change-the-title-name.png)
![Change name to Item Name](images/18-change-name-to-item-name.png)

> **LNG Translation:** the table sorts by **Vessel Name** rather than `Item Name`, and the renamed title column reads **"Vessel Name"** in the header — the label a duty officer scanning fifty rows actually needs to recognize a specific ship at a glance.

**What's shown:** switching to the Layout tab and setting the table's section to a Flex-2 column width.

![Click Layout icon](images/19-click-layout-icon.png)
![Change dimension to Flex, 2](images/20-change-dimension-to-flex-and-2.png)

> **LNG Translation:** unchanged mechanically — but on a real console, this is the moment an FDE decides how much horizontal room the cargo table needs relative to a berth-schedule or laycan-risk panel that will eventually sit beside it.

### Phase 3 — Build the Filter Panel

**What's shown:** adding a Filter List widget, pointing it at the same object set the table uses, and configuring a first filter.

![Add filter widget](images/21-click-the-plus-sign-to-add-widget-add-filter-widget.png)
![Add filter list](images/23-add-filter-list.png)
![Add the Order object set to the filter list](images/24-add-the-order-object-set-to-the-filter-list.png)
![Add filter option under Filter Config](images/25-add-filter-option-under-filter-config-list.png)
![Search for and add Item Name](images/26-serach-for-and-add-item-name.png)

> **LNG Translation:** the filter panel here would search for and add **Vessel Name**, **Berth**, **Cargo Status**, and **Offtaker/Charterer** — the four fields a terminal duty officer actually filters by when triaging "which cargoes need attention this shift." As in the main [Workshop Apps](https://github.com/manuelbomi/palantir-foundry-workshop-apps) repo, this filter must still be wired into a *derived* object set and plugged back into the table before it does anything — the mechanic doesn't change just because the object does.

## Roadmap: What a Production LNG Console Adds Next

The 26 screenshots above capture the module, the cargo table, and the first filter — the foundation. A terminal-ready console builds three things on top, following the same widget playbook demonstrated with charts and Actions in the main series ([Workshop Apps](https://github.com/manuelbomi/palantir-foundry-workshop-apps) and the [FDE Playbook](https://github.com/manuelbomi/palantir-foundry-fde-playbook)):

| Addition | Widget | Why it matters in LNG specifically |
|---|---|---|
| **Laycan Risk bar chart** | Chart: XY, segmented by Cargo Status, X-axis = Days Until Laycan Expiry | The single view that visually surfaces which cargoes are sliding toward a demurrage-triggering delay — the direct analogue of the "Days Until Due" bar chart in the main series, but here every bar represents real financial exposure |
| **Berth Utilization pie chart** | Chart: Pie, grouped by Berth | Shows at a glance whether berth capacity is the bottleneck behind a scheduling conflict |
| **`Reassign Marine Operator` / `Flag Boil-Off Risk` Actions** | Button Group + Ontology Actions | Governed, audited write-back — a duty officer reassigns a cargo or flags a BOG anomaly from the same screen, with the change permissioned and logged exactly as designed in the [FDE Playbook](https://github.com/manuelbomi/palantir-foundry-fde-playbook)'s Actions section |

## Design Decisions an FDE Has to Defend

- **Why keep the exact same widget sequence instead of a bespoke build?** Because the fastest way to convince a skeptical customer engineering team that Foundry isn't vaporware is to show them, live, that the identical low-code mechanics they can learn in an afternoon are what's actually running behind a mission-critical console — not a hidden layer of custom code.
- **Why translate field names instead of literally rebuilding on LNG data for this repo?** Because the goal here is to make the *transferability* of the skill undeniable — this repo is proof that an FDE who's internalized this widget sequence once can point it at chartering and marine-ops data on day one of an LNG engagement, not proof of a specific customer's cargo data (which would never be shareable publicly anyway).
- **Why prioritize the filter panel before charts, in an LNG build?** Because a duty officer's first move during a shift handover is almost always "show me only what's relevant to my berth/train" — filtering is the highest-value widget before any visualization, in LNG exactly as it was in the retail order console.

## Beyond LNG: The Same Console Pattern in Other Energy Verticals

Swap `Cargo` again and the identical widget sequence stands up the operational core of:

- **Pipeline operations** — a `Pipeline Segment` object, filtered by pressure-anomaly status, with a leak-risk bar chart
- **Power grid operations** — an `Outage` object, filtered by feeder and severity, with a restoration-time bar chart
- **Renewables asset management** — a `Turbine` or `Solar Array` object, filtered by site and health status, feeding the same reliability program bp is already running on Foundry
- **Upstream well operations** — a `Well` object, filtered by field and production status, powering the kind of "what configuration of wells and routing maximizes production" analysis Foundry already runs for energy supermajors

## Why Enterprises in Energy Choose Palantir

- **The platform already speaks the language of physical assets.** Foundry's Ontology and Vertex digital-twin tooling are purpose-built to represent trains, tanks, vessels, wells, and turbines as governed Objects — not an afterthought bolted onto a generic BI tool.
- **It closes the loop from sensor to action.** bp's reliability program is the proof point: over two million sensors feeding one integrated operating picture that engineers actually act from, not just monitor.
- **One platform scales across the whole energy transition.** The same bp partnership that started in upstream reliability has since extended into wind, solar, and EV charging — evidence that the pipeline → Ontology → app pattern this whole series demonstrates isn't specific to one energy sub-sector; it's the same chain, pointed at a different asset.
- **It's built for the person who has to make it real on-site.** An FDE embedded with a terminal's marine operations team, fluent in both Foundry's tooling and the vocabulary of laycans and boil-off gas, is what turns "we bought a platform" into "we stopped missing laycans."

## Repo Contents

```
├── images/     # 26 real, first-hand screenshots of the module build, in build order
└── README.md
```

## Related Repositories

| Repo | Focus |
|---|---|
| [palantir-foundry-data-pipelines](https://github.com/manuelbomi/palantir-foundry-data-pipelines) | Data integration with Pipeline Builder (Part 1) |
| [palantir-foundry-ontology](https://github.com/manuelbomi/palantir-foundry-ontology) | Modeling a dataset as a live Ontology Object (Part 2) |
| [palantir-foundry-workshop-apps](https://github.com/manuelbomi/palantir-foundry-workshop-apps) | The full operational dashboard build, all 47 steps (Part 3) |
| [palantir-foundry-fde-playbook](https://github.com/manuelbomi/palantir-foundry-fde-playbook) | The end-to-end case study and generalized FDE playbook (Part 4, capstone) |
| **palantir-foundry-lng-operations** (this repo) | The same Workshop mechanics, translated for LNG shipping and terminal operations |

---

*Author: [manuelbomi](https://github.com/manuelbomi) — built while working hands-on through Palantir's official Foundry training environment, reframed around how a Forward-Deployed Engineer would adapt the same skill set for the LNG and broader energy industry.*
