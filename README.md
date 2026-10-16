# MSYS2 AUR Package Migrator Template

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![License: AGPL v3](https://img.shields.io/badge/License-AGPL_v3-blue.svg)

A GitHub template repository designed to automatically fetch, patch, and build [ArchLinux/AUR](https://aur.archlinux.org/) packages for [MSYS2](https://www.msys2.org/) environments using GitHub Actions.

This template eliminates the manual overhead of porting AUR packages to Windows by providing a structured workflow that handles upstream updates, applies MSYS2-specific patches, and builds packages across multiple MSYS2 toolchains.

## ✨ Features

- **Multi-Environment Build Matrix:** Automatically builds packages for `mingw64`, `mingw32`, `ucrt64`, `clang64`, and `clangarm64`.
- **Automated Patching:** Apply MSYS2-specific patches to upstream AUR packages without maintaining a hard fork.
- **Custom Hooks:** Support for `pre-patch-msys2.sh` and `post-patch-msys2.sh` scripts for complex build logic.
- **Daily Upstream Checks:** Scheduled GitHub Actions run daily to detect upstream changes and rebuild.
- **Native MSYS2 Tooling:** Uses `makepkg-mingw` and sets `MINGW_PACKAGE_PREFIX` dynamically for each target environment.

## 📂 Repository Structure

To use this template effectively, your repository should follow this structure:

```text
.
├── .github/
│   └── workflows/
│       └── build.yml             # The CI/CD pipeline
├── origin/                       # Git submodule pointing to the upstream AUR/Arch repo
│   ├── PKGBUILD
│   └── ... (upstream source files)
├── patch/                        # MSYS2-specific patches (mirrors origin/ structure)
│   ├── PKGBUILD.patch
│   └── src/
│       └── main.c.patch
├── pre-patch-msys2.sh            # (Optional) Run before patches are applied
├── post-patch-msys2.sh           # (Optional) Run after patches are applied
└── README.md
```

### Directory & File Details

- **`origin/`**: This must be a Git submodule pointing to the upstream ArchLinux or AUR repository. The CI pulls the latest changes from this submodule on every run.
- **`patch/`**: Contains `.patch` files. The CI will apply these patches to the corresponding files in `origin/`. For example, `patch/src/main.c.patch` will patch `origin/src/main.c`.
- **`pre-patch-msys2.sh`**: An optional bash script executed in the MSYS2 environment *before* any patches are applied.
- **`post-patch-msys2.sh`**: An optional bash script executed in the MSYS2 environment *after* patches are applied and files are copied to the root.

## 🚀 Getting Started

1. **Create a new repository** using this template.
2. **Add the upstream AUR repository as a submodule** to the `origin/` directory:
   ```bash
   git submodule add https://aur.archlinux.org/<package-name>.git origin
   git commit -m "Add upstream AUR submodule"
   ```
3. **Create patches** (if necessary). If the upstream `PKGBUILD` or source code needs modifications for MSYS2, create a patch file and place it in the `patch/` directory, mirroring the upstream structure.
4. **(Optional) Add custom scripts**. Create `pre-patch-msys2.sh` or `post-patch-msys2.sh` in the root directory and make them executable (`chmod +x`).
5. **Push your changes**. The GitHub Actions workflow will automatically trigger and begin building your packages.

## ⚙️ How the CI/CD Works

The workflow (`.github/workflows/build.yml`) triggers on:
- Push to the `main` branch.
- A daily cron schedule (to catch upstream updates).
- Manual dispatch via the GitHub Actions UI.

### Build Pipeline Steps

1. **Checkout:** Recursively checks out the repository and the `origin/` submodule.
2. **MSYS2 Setup:** Initializes the MSYS2 environment for the specific target in the matrix and installs required build tools (`base-devel`, `git`, `patch`, and the appropriate toolchain).
3. **Patch Application:**
   - Runs `pre-patch-msys2.sh` (if present).
   - Applies all `.patch` files from `patch/` to `origin/` using `git apply` or `patch`.
   - Copies unpatched files from `origin/` to the repository root.
   - Runs `post-patch-msys2.sh` (if present).
4. **Build:** Executes `makepkg-mingw -sCLf --noconfirm`. The `MINGW_PACKAGE_PREFIX` environment variable is dynamically set based on the current matrix target (e.g., `mingw-w64-x86_64`).
5. **Upload Artifacts:** The resulting `*.pkg.tar.zst` files are uploaded as GitHub Actions artifacts.

## 📄 License

This project is licensed under the **GNU Affero General Public License v3.0 (AGPL-3.0)**.

This means you are free to use, modify, and distribute this template. However, if you modify the code and run it as a network service (e.g., hosting a public build server), you **must** make your modified source code available to the users of that service under the same AGPL license.

See the [LICENSE](LICENSE) file for the full license text.

## 🤝 Contributing

Contributions are welcome! If you find a bug or have a suggestion to improve this template, please open an issue or submit a pull request. 

1. Fork the repository.
2. Create your feature branch (`git checkout -b feature/AmazingFeature`).
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`).
4. Push to the branch (`git push origin feature/AmazingFeature`).
5. Open a Pull Request.
