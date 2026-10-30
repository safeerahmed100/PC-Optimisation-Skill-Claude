# Anti-cheat and FACEIT safety

Anti-cheat clients run at the kernel level and have opinions about the system's security and driver state. Breaking them can mean being unable to queue, kicked from matches, or, in the worst case, flagged. Check this file before any security, driver, or boot-configuration change.

## Contents
- Rule: verify current requirements
- Typical constraints
- Changes that need a compatibility check
- Safe-by-default behavior
- If the user can't queue after a change

## Rule: verify current requirements

FACEIT's requirements have changed over time and will change again. Do not rely on this file or on memory for exact requirements. When a decision depends on them (Secure Boot, TPM, VBS/Memory Integrity, test signing, specific drivers), search the web for FACEIT's current anti-cheat requirements and troubleshooting page, and quote what it says in your own words. If you can't verify, say so and choose the conservative option.

## Typical constraints (verify before relying on them)

- Secure Boot and TPM 2.0 have been required for some Windows versions or play modes. Don't recommend disabling either for performance reasons.
- Kernel-level anti-cheat can refuse to run alongside certain drivers (vulnerable or unsigned drivers, some hardware-monitoring or overclocking driver stacks, some virtualization/hypervisor setups). Be cautious with third-party tools that load kernel drivers.
- Test-signing mode, disabled driver signature enforcement, and modified Windows builds are generally incompatible.
- Hypervisor/virtualization settings and Memory Integrity can interact with anti-cheat; verify current guidance before changing them.

## Changes that need a compatibility check first

- Disabling or enabling VBS/Memory Integrity, Hyper-V, or virtualization features
- Any `bcdedit` change (test signing, boot options, timer settings)
- Installing or removing kernel-level utilities (hardware monitors, RGB/fan/overclock suites, driver-level tweak tools)
- Modified Windows images, driver-signature or secure-boot changes
- Third-party "optimizers" and registry tools that load services or drivers
- Overlays and injection-based tools (some are allowed, some flagged; verify per tool)

## Safe-by-default behavior

- Keep Secure Boot, TPM, driver signature enforcement, and Windows security features enabled.
- Use well-known, widely used monitoring tools, and note that some load drivers; mention to verify before leaving them running during matches.
- Run HWiNFO/CapFrameX/PresentMon for benchmarking, then close them if the anti-cheat guidance suggests it.
- If a tweak's anti-cheat compatibility can't be verified, don't recommend it for a FACEIT account the user cares about.

## If the user can't queue after a change

1. Revert the most recent change first (the change log tells you what it was).
2. Reboot and re-check FACEIT's troubleshooting page for the exact error message.
3. Restore from the restore point or registry backup if the revert doesn't fix it.
4. Re-run the audit's security block to confirm Secure Boot/TPM/VBS state matches what it was before.
5. Record which change caused it so it goes in the "Rejected" list.
