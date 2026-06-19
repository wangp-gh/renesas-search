# Test Scenarios — Quick Start Demo

This file contains three runnable test scenarios. After cloning the skill, try asking your OpenClaw agent each of these questions and see how it responds.

The three scenarios cover the three core guarantees of this skill:

| Scenario | What it tests |
|----------|---------------|
| **A. Trigger** | "Renesas-only" trigger rule — should activate only when Renesas is mentioned |
| **B. No fabrication** | Strict data sourcing — answer must cite datasheet PDF page, not training memory |
| **C. Typ-only reporting** | Honest disclosure when Min/Max are not specified in the datasheet |

---

## Scenario A — Trigger Rule Verification

**A.1 — Should trigger** (Renesas explicitly mentioned):

> "Compare DA14594 and DA14592 for a smart lock application. Which is more power-efficient?"

Expected behaviour:
- Skill activates
- Agent downloads both datasheets to `<cwd>/embedded_dev/renesas/datasheet/`
- Agent extracts and compares Rx/Tx current from the datasheet tables
- Recommends a chip based on actual datasheet values

**A.2 — Should NOT trigger** (no Renesas mention):

> "I need a BLE chip for a smart lock. What do you recommend?"

Expected behaviour:
- Skill does **not** activate
- Agent asks which vendor(s) to consider (Nordic, TI, ST, ESP, Renesas, …)
- The reference file `references/product_families.md` is not consulted

**A.3 — Should trigger** (product family named):

> "What's the difference between RL78/G23 and RL78/G14?"

Expected behaviour:
- Skill activates
- Both product pages verified
- Datasheets downloaded for both
- Comparison built from datasheet, not memory

---

## Scenario B — No-Fabrication Rule

**B.1** — The model must NOT answer from memory:

> "What is the hibernation current of DA14594?"

❌ **Wrong answer** (from memory / "reasonable estimate"):

> "DA14594 hibernation current: ~0.5 µA"

✅ **Correct answer** (from datasheet, with page citation):

> "DA14594 hibernation current: **Typ: 90 nA** (Min/Max: not specified in the electrical characteristics tables reviewed). Source: DA14594 product page Features list (https://www.renesas.com/da14594) and confirmed in datasheet PDF Table 6 (DC Characteristics, page 43)."

**B.2** — The model must admit gaps:

> "What is the BLE 5.4 throughput of DA14594?"

✅ **Correct answer**:

> "DA14594 is qualified against Bluetooth Core 5.3, not 5.4. The product page does not specify a 'throughput' number. The relevant spec is 'Wireless Standard: BLE 5.3 Core specification' (source: https://www.renesas.com/da14594 Parameters table). Maximum application throughput depends on connection interval, MTU, and packet size — these are stack parameters, not part specifications."

**B.3** — The model must cite the page, not "P.1":

> "What is the Rx sensitivity of DA14592?"

✅ **Correct answer** (from Renesas product page Parameters table):

> "DA14592 Rx sensitivity: **Typ: −97 dBm** (Min/Max: not specified). Source: https://www.renesas.com/da14592 Parameters table."

❌ **Wrong answer** (cited the wrong source):

> "−97 dBm (from P.1)" — P.1 is the overview/description; not a specification.

---

## Scenario C — Typ-only Reporting

**C.1** — Renesas datasheet often has blank Min/Max:

> "What is the Tx current of DA14594 at 0 dBm, 3 V, DCDC on?"

✅ **Correct answer** (qualifies Typ, notes missing Min/Max):

> "DA14594 Tx current: **Typ: 2.3 mA** (Min/Max: not specified in the electrical characteristics tables). Conditions: 0 dBm, VBAT = 3 V, DCDC on. Source: Renesas product page Parameters table (https://www.renesas.com/da14594) and confirmed in datasheet PDF Table 6 / DC Characteristics, page 43+."

❌ **Wrong answer** (treats Typ as if it were a full spec):

> "Tx current: 2.3 mA at 0 dBm" — implies a guaranteed spec, which it is not.

**C.2** — Asking for Min when datasheet doesn't publish it:

> "What is the minimum Rx current of DA14594?"

✅ **Correct answer** (honest about the gap):

> "The DA14594 datasheet's electrical characteristics tables publish only Typ values for Rx current (Typ: 1.2 mA). Min and Max are not specified. The data should not be inferred from a similar product (e.g., DA14592) because Min/Max depend on the specific chip design and process corner, not on the family."

---

## Verifying the Skill Worked

After running any scenario, check the following:

1. **Datasheet files exist**:

   ```bash
   ls -la embedded_dev/renesas/datasheet/
   ```

   You should see `<PartNumber>_datasheet.pdf` for every part mentioned.

2. **No fabricated numbers**:

   - If the agent states a numerical value (e.g., "1.2 mA"), it must cite:
     - A specific PDF page (e.g., "PDF page 43, Table 6"), OR
     - The Renesas product page URL (e.g., "https://www.renesas.com/da14594 Parameters table").
   - Single numbers without a source = fabrication.

3. **Typ/Min/Max discipline**:

   - Values that the datasheet publishes only as Typ must be qualified.
   - "Min/Max: not specified" notes must appear when applicable.

4. **Trigger discipline**:

   - Generic "find a BLE chip" questions should NOT activate the skill.
   - The skill activates only when the user says "Renesas", "瑞萨", or names a Renesas product family.

---

## If Something Fails

Common failure modes and how to report them:

| Symptom | Likely cause |
|---------|--------------|
| Skill activates on "find a BLE chip" | Trigger rule not enforced; check `SKILL.md` description |
| Numerical answer with no source | No-fabrication rule not followed; check `## Absolute Rule` section |
| Typ value reported as if it were a full spec | Typ-only reporting rule not followed; check `### Typ-only Reporting` section |
| Datasheet not downloaded | `cwd/embedded_dev/renesas/datasheet/` not writable, or `web_fetch`/`curl` failed |
| Skill not visible to your agent | `~/.openclaw/workspace/skills/renesas-search/SKILL.md` not present, or `name: renesas-search` missing in frontmatter |

Open an issue on https://github.com/wangp-gh/renesas-search with the failing scenario letter (A/B/C) and the agent's output.
