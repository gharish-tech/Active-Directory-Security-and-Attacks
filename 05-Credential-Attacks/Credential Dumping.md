### LSASS Protection Baseline

The LSASS process was confirmed to be running on Windows Server 2022 Standard Evaluation (OS build 20348).

Microsoft Defender antivirus and real-time protection were enabled. However, the `RunAsPPL` registry value was not present in the queried registry path, and the attempted Device Guard and WinInit event queries did not return evidence confirming LSASS protected-process mode.

**Conclusion:** LSASS protection status could not be confirmed from the available checks. The absence of a registry value or matching event does not, by itself, prove that protection is disabled.

**Next steps:** If required, verify the applicable Windows security configuration and runtime protection status using version-appropriate Microsoft documentation.

