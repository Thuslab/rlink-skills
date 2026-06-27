# ELF Symbol Analysis Tool Schema

> Call tools via `rlink_invoke({ "tool": "...", "args": {...} })`.
> `?` suffix means optional.

## elf_addr2line

Resolve addresses to source. addresses: [u64]. Returns {locations: [{file, line, function}]}. [parallel-safe]

**Parameters**

- `addresses` (array) — List of virtual addresses (hex or decimal) to resolve.
- `elf_path` (string?) — Path to a loaded ELF file. Omit to use the only loaded ELF.

---

## elf_disasm

Disassemble instructions. address: "0x...", count: default 20. Returns {instructions: [{addr, mnemonic, operands}]}. [parallel-safe]

**Parameters**

- `address` (string) — Start address. Hex (e.g. "0x08000000") or decimal string.
- `count` (integer?) — Number of instructions to disassemble (default 20, max 200).
- `elf_path` (string?) — Path to a loaded ELF file. Omit to use the only loaded ELF.

---

## elf_load

Load ELF/AXF for symbol lookup. Returns {arch, entry, symbol_count}. Sets as default ELF. [serialized-global]

**Parameters**

- `path` (string) — Absolute or relative path to the ELF/AXF file.

---

## elf_symbols

Query symbol table. filter: name substring. Returns {symbols: [{name, address, size}], total, has_more}. [parallel-safe]

**Parameters**

- `elf_path` (string?) — Path to a loaded ELF file. Omit to use the only loaded ELF.
- `filter` (string?) — Substring filter for symbol names.
- `limit` (integer?) — Maximum number of results (default 100, max 1000).

---

