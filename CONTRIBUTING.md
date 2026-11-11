# Developing your fork

## Git remotes

For this fork, `origin` should point to
`https://github.com/I-M-AJ/Advanced-RISC-V-32-bit-SoC.git`.
The original repository can be kept as `upstream` for fetching changes:
`https://github.com/NguyenToQuocViet/Advanced-RISC-V-32-bit-SoC.git`.

The following local settings make ordinary pushes target your fork and disable
pushes through the `upstream` remote:

```powershell
git config --local remote.pushDefault origin
git config --local push.default simple
git config --local remote.upstream.pushurl DISABLED
git remote -v
```

These settings are local to this checkout and are not included in commits.
They do not prevent someone from explicitly pushing to a different URL.
Fetching from `upstream` does not modify the author's repository.

Create a branch before starting a feature:

```powershell
git switch -c my-feature
git add -- path-to-your-new-file
git diff --cached
git commit -m "Describe your change"
git push -u origin my-feature
```

Replace the example branch and file names as appropriate. Git tracks files, not
empty directories. Add a real file or a `.gitkeep` placeholder to retain an
otherwise empty directory. If a file is unexpectedly hidden, check it with
`git check-ignore -v -- path-to-your-new-file`.

New source files in `work` and `project_*` directories are allowed. Generated
build, simulator, and Vivado artifacts remain ignored; do not commit them.
Preserve the existing license and copyright notices.

## Running tests on Windows with WSL

The root [Makefile](Makefile) uses a POSIX shell and utilities. Use Ubuntu WSL
rather than the Windows GnuWin32 `make` executable.

From PowerShell in the repository root, enter WSL:

```powershell
wsl -d Ubuntu-24.04
```

WSL normally inherits the current directory. Confirm that `pwd` and `ls Makefile`
identify this checkout before running the following Linux commands.

The simulation prerequisites are GNU Make, a C++ compiler, Verilator with
`--binary` and `--timing` support, and standard shell utilities including `grep`
and `tee`. The RISC-V test images additionally require Python 3 and the RISC-V
cross-toolchain:

```bash
sudo apt-get update
sudo apt-get install build-essential verilator python3 gcc-riscv64-unknown-elf binutils-riscv64-unknown-elf
make test TARGET=fpga7 TEST=soc_addr_decode
```

Before running core or SoC tests, generate their memory images using the
[test-image Makefile](tb/riscv_test/Makefile):

```bash
make -C tb/riscv_test
make lint TARGET=fpga7
make regression TARGET=fpga7
```

Use the registered test names and profile-specific regression lists in the root
[Makefile](Makefile). See [AGENTS.md](AGENTS.md) for the validation requirements
for changes affecting multiple profiles. Put generated validation logs under
`build/`.

Icarus Verilog is not a substitute for the canonical Verilator test targets.
Vivado is only needed for FPGA synthesis. LibreLane, Docker, and the Sky130 PDK
are needed for the separate Sky130 flow, not ordinary simulation tests.
The [software Makefile](sw/Makefile) also uses `xxd` and `awk` when generating
software images.
