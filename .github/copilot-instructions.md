# GameBoost Pro project instructions

## Project scope

- This is a Windows 10/11 x64 desktop application built with C++20, Qt 6.6 or newer, and QML.
- Use CMake with the vcpkg manifest in the repository root. Configure with the vcpkg toolchain file and an x64 Windows toolchain.
- Link shared application dependencies through the `gameboost_pro_dependencies` interface target.
- There is no executable target yet. When adding one, link it to `gameboost_pro_dependencies` and call `gameboost_apply_app_manifest(<target>)`; this currently embeds the manifest for MSVC-compatible linkers.
- The project is being scaffolded in stages. Do not invent missing modules or describe unimplemented functionality as working.

## Implementation practices

- Keep UI and presentation in QML; put application logic and platform integrations in C++.
- Prefer Qt APIs for portable UI and application services. Isolate direct Win32 calls behind small C++ interfaces and use them only where needed.
- Treat system optimization as a user-visible, opt-in operation. Explain changes before applying them, validate inputs, report errors, and provide a safe way to restore settings where applicable. Do not silently change registry, services, startup configuration, network settings, or security controls.
- The application manifest requests administrator privileges. Do not use elevation as a reason to bypass consent, Windows protections, or error handling.
- Use `libsodium` Argon2id APIs for password-based key derivation; never implement cryptographic primitives yourself. Use SQLite through its imported CMake target, `nlohmann_json` for JSON, and `spdlog` for logging.
- Keep Win32 handles and resources under RAII ownership. Check API return values and surface actionable errors; avoid silent fallbacks and broad exception swallowing.
- Keep CMake dependency discovery and target linking explicit. Add new dependencies to both `vcpkg.json` and the CMake configuration when required.
- Do not add credentials, machine-specific paths, or generated build artifacts to source control.

## Verification

- Build with a 64-bit Windows toolchain after configuring dependencies through vcpkg.
- Add focused tests for new logic, especially code that changes system state; tests must not modify the host machine's settings.
- Run the narrowest relevant build or test checks and report any unavailable Windows-only validation plainly.
