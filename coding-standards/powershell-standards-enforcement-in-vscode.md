


# ✅ PowerShell Standards Enforcement in Visual Studio Code

## 📁 Folder Structure

```text
<your-repo>/
├── .vscode/
│   ├── settings.json
│   └── PSScriptAnalyzerSettings.psd1
├── .git/
│   └── hooks/
│       └── pre-commit   # Git hook for standards enforcement
```

---

## ⚙️ VSCode `.vscode/settings.json`

```json
{
    "editor.tabSize": 4,
    "editor.insertSpaces": true,
    "editor.rulers": [120],
    "editor.formatOnSave": true,
    "editor.wordWrap": "off",
    "files.trimTrailingWhitespace": true,
    "files.insertFinalNewline": true,
    "powershell.enableProfileLoading": true,
    // https://learn.microsoft.com/en-us/powershell/utility-modules/psscriptanalyzer/using-scriptanalyzer?view=ps-modules
    "powershell.scriptAnalysis.enable": true,
    "powershell.scriptAnalysis.settingsPath": ".vscode/PSScriptAnalyzerSettings.psd1",
    "powershell.codeFormatting.openBraceOnSameLine": true,
    "powershell.codeFormatting.newLineAfterOpenBrace": true,
    "powershell.codeFormatting.newLineAfterCloseBrace": true,
    "powershell.codeFormatting.whitespaceAroundOperator": true,
    "powershell.codeFormatting.useCorrectCasing": true,
    "powershell.codeFormatting.whitespaceBeforeOpenBrace": true,
    "powershell.codeFormatting.trimWhitespaceAroundPipe": true,
    "powershell.codeFormatting.autoCorrectAliases": true,
    "editor.codeActionsOnSave": {
        "source.organizeImports": true,
        "source.fixAll": true
    },
    "terminal.integrated.defaultProfile.windows": "PowerShell"
}
```

---

## 📜 `.vscode/PSScriptAnalyzerSettings.psd1`

```powershell
@{
    ExcludeRules = @('PSAvoidUsingWriteHost')

    Rules = @{
        "PSAvoidUsingCmdletAliases"        = @{ Enable = $true }
        "PSUseCmdletCorrectCasing"         = @{ Enable = $true }
        "PSUseApprovedVerbs"               = @{ Enable = $true }
        "PSUseConsistentWhitespace"        = @{ Enable = $false }
        "PSUseConsistentIndentation"       = @{ Enable = $true }
        "PSAvoidTrailingWhitespace"        = @{ Enable = $true }
        "PSAvoidUsingBackticks"            = @{ Enable = $true }
        "PSUseCorrectCasing"               = @{ Enable = $true }
        "PSUseSingularNouns"               = @{ Enable = $true }
        "PSProvideCommentHelp"             = @{ Enable = $true; Options = @{ Required = "ExportedFunctions" } }
        "PSAvoidUsingPositionalParameters" = @{ Enable = $true }
        "PSAvoidLongLines"                 = @{ Enable = $true; Options = @{ MaximumLineLength = 120 } }
        "PSAvoidGlobalVars"                = @{ Enable = $true }
    }
}
```

---

## 🪝 Git Hook: `.git/hooks/pre-commit` (Windows)

```powershell
pwsh -NoProfile -Command {
    $files = git diff --cached --name-only --diff-filter=ACM | Where-Object { $_ -like '*.ps1' }
    $failed = $false
    foreach ($file in $files) {
        $issues = Invoke-ScriptAnalyzer -Path $file -Settings '.vscode/PSScriptAnalyzerSettings.psd1'
        if ($issues) {
            Write-Host "`nStandards violations found in $file" -ForegroundColor Red
            $issues | Format-Table
            $failed = $true
        }
    }
    if ($failed) {
        exit 1
    }
}
```

Ensure it's saved as UTF-8 and run:

```powershell
git update-index --chmod=+x .git/hooks/pre-commit
```

---

## 🔄 Git Line Ending Config: `core.autocrlf=input`

```bash
git config --global core.autocrlf input
```

### 🔍 Explanation:

* **input**: Git converts CRLF to LF when committing but leaves LF unchanged on checkout.
* Best for: **Windows devs working with Unix-style repos** (like PowerShell modules shared on GitHub).
* Prevents: `pre-commit` hooks from breaking due to Windows-style line endings (CRLF).

---

## 🧭 PSScriptAnalyzer Rule Coverage

| Standard Category               | Covered by PSSA? | Notes                                                                |
| ------------------------------- | ---------------- | -------------------------------------------------------------------- |
| 1. Function Structure           | ❌                | Manual review required (Begin/Process/End, CmdletBinding)            |
| 2. Comment-Based Help           | ✅ Partial        | `PSProvideCommentHelp` enforces help block on exported functions     |
| 3. Parameter Block              | ✅ Partial        | Covers casing, aliases, positional use, but not full Validate chains |
| 4. Naming                       | ✅                | `PSUseApprovedVerbs`, `PSUseSingularNouns`, `PSUseCorrectCasing`     |
| 5. Module Imports               | ❌                | Must be enforced via templates or reviews                            |
| 6. TLS Enforcement              | ❌                | No automated check for TLS line                                      |
| 7. Azure DevOps Logging         | ❌                | Use of `Write-Host` allowed manually for pipeline logs               |
| 8. Variable Initialization      | ❌                | Manual (Begin block usage, password generation)                      |
| 9. Secure Strings & Credentials | ❌                | Manual only                                                          |
| 10. SecretStore Operations      | ❌                | Manual only                                                          |
| 11. PSResource Repository       | ❌                | Manual only                                                          |
| 12. Module Installation         | ❌                | Manual enforcement for splatting + logging                           |
| 13. Error Handling              | ❌                | Try/catch + `-ErrorAction` not enforced via PSSA                     |
| 14. Splatting                   | ✅                | `PSAvoidUsingBackticks` helps enforce splatting                      |
| 15. Inline Comments             | ❌                | Manual review                                                        |
| 16. String Formatting           | ❌                | Manual (`"` vs `'`, URL breaking)                                    |
| 17. Output Verbosity            | ❌                | Manual (Write-Verbose for info vs Write-Host)                        |
| 18. Finalization                | ❌                | Manual (`Write-Verbose 'Initialization complete.'`)                  |
| 19. Script Invocation Guard     | ❌                | Manual (`if ($MyInvocation...)`)                                     |
| 20. General Style               | ✅ Partial        | Whitespace, indentation, line length                                 |
| 21. Script Prerequisites        | ❌                | No enforcement of `#Requires`                                        |

---

## ✅ Summary

* **VSCode & PSSA**: Catches naming, spacing, backticks, casing, comment help.
* **Git hook**: Blocks non-compliant commits.
* **Manual review** still needed for logic, TLS, credential handling, and modular structure.

To extend further, consider writing custom fixers or using templates for new scripts.
