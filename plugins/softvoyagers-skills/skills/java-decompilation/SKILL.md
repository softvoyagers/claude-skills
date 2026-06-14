---
name: java-decompilation
description: |
  Use when the user wants to decompile, reverse-engineer, or read compiled
  Java — `.class` files, a `.jar`/`.war`/`.aar`, or classes extracted from one —
  back into readable `.java` source, and optionally save that source into a
  repository to study or diff it.
  Covers tool install (Vineflower + CFR), inspecting bytecode, decompiling a
  whole jar or directory to a source tree, restricting to one package, and
  cross-checking a single class when output looks wrong.
  Trigger with phrases like "decompile this jar", "decompile java class",
  "reverse engineer this .class", "turn these classes into source",
  "read the bytecode", "what does this compiled class do",
  "extract source from a jar".
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
version: 1.0.0
license: MIT
---

# Java Decompilation

## Overview
Recover readable `.java` source from compiled Java bytecode (`.class` files, jars,
or classes already extracted from a jar). Two complementary tools are used:

- **Vineflower** — the maintained successor to Fernflower/Quiltflower. **Primary tool.**
  Best readability, decompiles a whole jar or directory tree in one command, can
  restrict output to a single package. Use it for everything by default.
- **CFR** — the most robust engine on hostile/obfuscated bytecode. **Cross-check tool.**
  When a Vineflower method body looks wrong or implausible, re-run *that one class*
  through CFR and compare. CFR often picks cleaner variable names and inverts guard
  clauses for readability.

Output quality note: decompilation recovers logic, control flow, constants, and
string/enum references faithfully. It does **not** recover original local-variable
names, comments, or names that were stripped by an obfuscator — single-letter
class/field/method names in the output are the *original obfuscation*, not a tool
failure. Generic types and most field/constant values come back intact.

## Prerequisites
- macOS with Homebrew (`brew`). Linux: use the GitHub release jars (see Resources)
  with a JRE 17+ and `java -jar`.
- The decompiler tools, installed in one command (Homebrew pulls a JDK automatically
  as a dependency — you do **not** need to install Java separately):

```bash
brew install vineflower cfr-decompiler
```

Verify:

```bash
vineflower --help        # prints usage
cfr-decompiler --version # prints e.g. "CFR 0.152"
```

> If you only have raw jars (no brew): `java -jar vineflower-<ver>.jar ...` and
> `java -jar cfr-<ver>.jar ...`. Vineflower needs **JRE 17+**; CFR runs on JRE 8+.

## Instructions

### Step 1: Identify the input and pick a destination

Inputs can be a `.jar`/`.war`/`.aar`, a single `.class`, or a directory of extracted
`.class` files. Decide where the `.java` should land — a scratch dir for reading, or
a path inside a repo if the user wants the source saved/committed.

```bash
# Inspect what's inside a jar before decompiling (optional):
unzip -l target.jar | grep '\.class$' | head
# Or inspect one class's signatures/bytecode without decompiling (JDK 'javap'):
javap -p -c -classpath target.jar com.example.Foo | head -40
```

### Step 2: Decompile the whole input (Vineflower — default)

Vineflower infers output from the path: a **folder** target ⇒ a package-structured
source tree; a `.jar` target ⇒ a source jar. The package directories are created
for you.

```bash
# Whole jar  ->  source tree
vineflower path/to/input.jar  ./decompiled/

# Directory of extracted .class files  ->  source tree
vineflower ./extracted-classes/  ./decompiled/
```

Useful flags on obfuscated or large inputs:

```bash
vineflower \
  --only=com/example/security \  # restrict OUTPUT to one package (slash form); pass the jar so context stays full
  --remove-synthetic=1 \         # drop synthetic/bridge members for cleaner reads
  --decompile-generics=1 \       # recover generic types (default on)
  --thread-count=8 \             # parallelism
  path/to/input.jar  ./decompiled/
```

Decompiling the **jar** while restricting output with `--only=` is better than
pointing at a pre-extracted package directory: Vineflower keeps the rest of the jar
on its classpath, so type inference is more accurate.

### Step 3: Save the source into a repository (when asked)

To "put the `.java` into a repo to use those files", target a path inside the repo
directly, then let the user review/commit:

```bash
vineflower --only=com/example/feature  app.jar  ./src/main/java/
git -C ./repo status        # show what landed; do NOT commit unless the user asks
```

Keep decompiled output in a clearly named directory (e.g. `decompiled/` or a
dedicated source root). Note for the user that decompiled source is derived,
may carry obfuscated names, and is for analysis — confirm before committing it
into a build path.

### Step 4: Cross-check a suspicious class (CFR)

If a specific decompiled method looks wrong, re-run just that class through CFR and
compare. CFR writes to **stdout** for a single class.

```bash
# Single class -> stdout (the per-class second opinion)
cfr-decompiler ./extracted-classes/com/example/Foo.class \
  --comments false --renameillegalidents true

# Whole jar, restricted to a package, written to a tree.
# NOTE: CFR's --jarfilter regex matches the DOTTED class name, not the slash path:
cfr-decompiler app.jar \
  --jarfilter 'com.example.security.*' \
  --outputdir ./decompiled-cfr/ \
  --comments false --renameillegalidents true --silent true
```

`--renameillegalidents true` rewrites names that obfuscation made into illegal Java
identifiers; `--comments false` drops CFR's own banner/inline notes.

### Step 5: Read and analyze

Now treat the output as normal source: `Grep`/`Glob` for entry points, read the
key classes, follow the call chain. For obfuscated trees, anchor on the readable
signals — string literals, enum/constant references, JDK/library API calls, and
exception types survive decompilation and reveal intent even when names don't.

## Error Handling

| Symptom | Cause | Solution |
|---------|-------|----------|
| `Unable to locate a Java Runtime` | No JDK on PATH | Install via brew formula (pulls a JDK), or `brew install openjdk` / `--cask temurin` for raw-jar use |
| Vineflower fails to start / class-version error | JRE older than 17 | Vineflower needs JRE 17+; use the brew formula's JDK or install `openjdk` (17+) |
| CFR `--jarfilter` produces 0 files | Regex written as a slash path | CFR matches the **dotted** FQN: use `com.example.pkg.*`, not `com/example/pkg/.*` |
| CFR "Is a directory" / `FileNotFoundException` | A directory was passed to CFR | CFR takes a jar or a single `.class`; for a package use the jar + `--jarfilter`, or feed Vineflower the directory instead |
| Single-letter class/field/method names | Original obfuscation, not a decompiler bug | Expected; analyze via strings/constants/API calls. Try CFR's `--renameillegalidents true` for legal but synthetic names |
| A method body looks implausible | Engine-specific decompile artifact | Cross-check that one class in the other tool (Step 4); trust the one whose control flow type-checks |
| Output missing some inner classes | Inner classes fold into the outer `.java` | Expected — `.java` count is lower than `.class` count; open the enclosing class |

## Resources
- [Vineflower repo](https://github.com/Vineflower/vineflower) · [releases](https://github.com/Vineflower/vineflower/releases) · [usage docs](https://vineflower.org/usage/)
- [CFR repo](https://github.com/leibnitz27/cfr) · [benf.org/cfr](https://www.benf.org/other/cfr/)
- Homebrew formulae: [`vineflower`](https://formulae.brew.sh/formula/vineflower) · [`cfr-decompiler`](https://formulae.brew.sh/formula/cfr-decompiler)
- `javap` (ships with any JDK) for signature/bytecode inspection without decompiling.
