# Security Advisory: Path Traversal (Zip Slip) in FacturaScripts

## Vulnerability Overview
- **Vulnerability Type:** Path Traversal / Zip Slip
- **Affected Component:** `Plugins::add()` function in `Plugins.php`
- **Discovery Credit:** ZeroXJacks
- **Status:** Patched by NeoRazorX

## Description
A path traversal vulnerability, commonly known as a **Zip Slip**, was identified in the plugin upload mechanism of FacturaScripts. The application failed to adequately sanitize the file paths of entries within uploaded ZIP archives. While the system attempted to validate that the archive contained only a single root directory, it did not account for relative path segments (`../`) within the filenames.

An attacker with administrative privileges could craft a malicious ZIP file containing entries such as `PluginName/../../shell.php`. During the extraction process, the operating system interprets the traversal sequences, allowing the file to be written outside the intended `plugins/` directory and into the web root or other sensitive locations.



## Technical Details
The vulnerability resided in the logic used to identify the root folder of a plugin. The code utilized `explode()` on the file path and only checked the first element:

```php
// Vulnerable logic
$path = explode('/', $data['name']);
if (count($path) > 1) {
    $folders[$path[0]] = $path[0];
}
```
Because the check only validated the uniqueness of the first segment ($path[0]), it could be easily bypassed. The subsequent extraction did not strip these traversal sequences, leading to Arbitrary File Write.
## Impact
Confidentiality: High. Attackers can read sensitive configuration files and database credentials.
Integrity: High. Attackers can overwrite or modify any file the web server has permissions to access.
Availability: High. Malicious actors could delete the installation or disrupt services.
Remote Code Execution (RCE): By overwriting or creating .php files in the web root, an attacker can achieve full command execution on the underlying server.
