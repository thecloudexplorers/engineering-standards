# PowerShell Coding Standards

*A concise guide outlining required conventions for PowerShell scripts within this repository.*

## 1. Function Structure

1. **1.1** Always write an advanced function, not inline code.
   *Note:* Use the `function` keyword with `[CmdletBinding()]` so your code supports common parameters.

   ```powershell
   function Get-Example {
       [CmdletBinding()]
       param() 
       process { "Hello" }
   }
   ```

2. **1.2** Use CmdletBinding() at top.
   *Note:* Placing `[CmdletBinding()]` enables `-Verbose`, `-ErrorAction`, etc., without extra code.

3. **1.3** Split into Begin, Process, and End blocks—even if empty.
   *Note:* Structure your logic so initialization goes in `Begin {}`, main work in `Process {}`, cleanup in `End {}`.

---

## 2. Comment-Based Help

1. **2.1** Include a `<# .SYNOPSIS … .DESCRIPTION … .PARAMETER … .EXAMPLE … #>` block above the function.
   *Note:* A proper help block allows users to run `Get-Help` with full context.

2. **2.2** Document every parameter with `.PARAMETER Name` and a clear description.
   *Note:* Ensures each `param()` entry is self-explanatory when viewed via help.

---

## 3. Parameter Block

1. **3.1** One attribute per line.
   *Note:* Improves readability and diff tracking.

2. **3.2** Always start with `[Parameter(...)]`.
   *Note:* Every parameter declaration must begin with its attribute.

3. **3.3** Use `[ValidateNotNullOrEmpty()]` immediately after if mandatory.
   *Note:* Ensures required parameters always have a value.

4. **3.4** Mandatory parameters: `[Parameter(Mandatory=$true)]`.
   *Note:* Prompts the user if they forget to supply that argument.

5. **3.5** Optional parameters: `[Parameter()]` and default value immediately after the name.
   *Note:* Makes it explicit which parameters are optional and their defaults.

6. **3.6** Declare the type in square brackets (e.g. `[string]`, `[int]`, `[string[]]`).
   *Note:* Strong typing helps catch errors early.

---

## 4. Naming

1. **4.1** Function names use PascalCase with a verb-noun pair (e.g. `Initialize-ArtifactFeed`).
   *Note:* Follows PowerShell’s Verb-Noun convention for discoverability.

---

## 5. Module Imports

1. **5.1** Load required modules in the Begin block.
   *Note:* Guarantees dependencies are available before processing.

2. **5.2** Use `Import-Module @params` splatting when three or more parameters.
   *Note:* Keeps commands tidy when specifying multiple options.

3. **5.3** Fail fast: `-ErrorAction Stop` and catch with try/catch, rethrow on failure.
   *Note:* Ensures module loading errors halt execution immediately.

---

## 6. TLS Enforcement

1. **6.1** In Process, set `[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12`.
   *Note:* Forces use of TLS 1.2 for all outbound web requests.

   ```powershell
   Write-Verbose "Enforcing TLS 1.2"
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   ```

2. **6.2** Wrap in Write-Verbose description.
   *Note:* Signals to the user that security settings are being applied.

---

## 7. Azure DevOps Logging

1. **7.1** Surround logical sections with `Write-Host "##[group]…"` and `Write-Host "##[endgroup]"`.
   *Note:* Collapses output in Azure Pipelines for clarity.

2. **7.2** Use clear section titles (e.g. “Prepare variables”, “Set secret vault and secret store”).
   *Note:* Helps operators quickly find relevant log segments.

---

## 8. Variable Initialization

1. **8.1** Static defaults declared in Begin.
   *Note:* Centralizes constant values for easy maintenance.

2. **8.2** Automatic values (like random passwords) generated in Begin, with inline comments explaining each step.
   *Note:* Ensures reproducible randomness and documents how values are derived.

---

## 9. Secure Strings & Credentials

1. **9.1** Always convert PAT or passwords via `ConvertTo-SecureString -AsPlainText -Force`.
   *Note:* Ensures secrets are handled as secure strings.

2. **9.2** Create PSCredential with `New-Object System.Management.Automation.PSCredential($user, $secureString)`.
   *Note:* Standard PSCredential objects work with cmdlets expecting credentials.

---

## 10. SecretStore Operations

1. **10.1** Unregister existing vault if present (`Get-SecretVault -ErrorAction SilentlyContinue`).
   *Note:* Cleans up stale vault registrations before proceeding.

2. **10.2** Register vault with `Register-SecretVault @params`.
   *Note:* Defines where secrets will be stored.

3. **10.3** Reset store via `Reset-SecretStore @params` splatting.
   *Note:* Clears and reinitializes the local secret store.

4. **10.4** Unlock via `Unlock-SecretStore @params`.
   *Note:* Makes the secret store available for read/write.

5. **10.5** Store secret via `Set-Secret @params`.
   *Note:* Persists credentials or tokens securely.

---

## 11. PSResource Repository

1. **11.1** Unregister existing repository if present.
   *Note:* Avoids conflicts when re-registering feeds.

2. **11.2** Register new repository with `Register-PSResourceRepository @params` splatting.
   *Note:* Adds custom module repositories for installations.

3. **11.3** Use `$CredentialInfo = [Microsoft.PowerShell.PSResourceGet.UtilClasses.PSCredentialInfo]::new(...)` to pass credentials.
   *Note:* Properly encapsulates credentials for PSResource operations.

---

## 12. Module Installation

1. **12.1** Loop through `$CustomModules` array.
   *Note:* Supports batch installation of modules.

2. **12.2** Install each with `Install-PSResource @params` splatting.
   *Note:* Uses parameter splatting for clarity.

3. **12.3** Log each install via `Write-Host`.
   *Note:* Provides feedback on progress.

---

## 13. Error Handling

1. **13.1** Every critical call uses `-ErrorAction Stop`.
   *Note:* Forces exceptions on failures rather than silent errors.

2. **13.2** Wrap module imports in try/catch and rethrow to fail the pipeline.
   *Note:* Ensures upstream errors propagate correctly.

---

## 14. Splatting

1. **14.1** For any call with more than two parameters, build a hashtable `$params = @{…}` then call `Cmdlet @params`.
   *Note:* Improves readability for cmdlets with many options.

2. **14.2** Never use backtick line continuation—always use splatting or a single line.
   *Note:* Avoids fragile and hard-to-read backtick breaks.

---

## 15. Inline Comments

1. **15.1** Comment any complex logic (password generation, URL composition, splatting blocks).
   *Note:* Helps future maintainers understand nontrivial code.

2. **15.2** Keep comments concise and above the line they describe.
   *Note:* Maintains a clean association between comment and code.

---

## 16. String Formatting

1. **16.1** Use double-quoted strings when interpolation is needed.
   *Note:* Ensures variable values are expanded inside strings.

2. **16.2** Break long URLs into concatenations or variables with inline comments.
   *Note:* Prevents overly long lines and clarifies URL segments.

---

## 17. Output Verbosity

1. **17.1** Use `Write-Verbose` for non-critical status messages.
   *Note:* Users can enable verbose mode for detailed output.

2. **17.2** Use `Write-Host` inside pipeline groups for high-level progress.
   *Note:* Ensures key milestones are always visible.

---

## 18. Finalization

1. **18.1** In End, emit a `Write-Verbose 'Initialization complete.'`.
   *Note:* Signals successful completion in verbose logs.

2. **18.2** Do not emit extra output on success.
   *Note:* Keeps success output clean for pipeline consumption.

---

## 19. Script Invocation Guard

1. **19.1** If the script file is executed directly (`if ($MyInvocation.MyCommand.Path -eq $PSCommandPath)`), invoke the function with `@PSBoundParameters`.
   *Note:* Prevents accidental execution when dot-sourced.

---

## 20. General Style

1. **20.1** Indent four spaces.
   *Note:* Consistent indentation improves readability.

2. **20.2** No trailing commas or semicolons.
   *Note:* Avoids syntax issues and lint warnings.

3. **20.3** Keep line length under \~120 characters.
   *Note:* Ensures code fits typical editor windows.

4. **20.4** One statement per line.
   *Note:* Simplifies debugging and version control diffs.

---

## 21. Script Prerequisites

1. **21.1** Use `#Requires` statements at the top to define environment prerequisites. Note: Use `#Requires -PSEdition Desktop` and `#Requires -Modules Az` to fail fast when conditions aren't met.

## References

* [Approved Verbs (Microsoft Learn)](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/approved-verbs-for-windows-powershell-commands)
* [Cmdlet & Script Development Guidelines](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/strongly-encouraged-development-guidelines)
* [PoshCode PowerShell Practice & Style](https://github.com/PoshCode/PowerShellPracticeAndStyle)
* [PowerShell Community Style Guide](https://learn.microsoft.com/en-us/powershell/scripting/community/contributing/powershell-style-guide)

2. **4.2** Use approved verbs in function names. Note: Use `Get-Verb` to select an approved verb (e.g., Get-UserAccount).

3. **4.3** Ensure function names clearly describe the action and the target. Note: Use specific PascalCase nouns (e.g., `Get-User` vs `Get-Users`).

4. **4.4** Use singular nouns for cmdlet names unless returning multiple items. Note: Prefer singular noun for commands returning a single object.

5. **3.7** Use validation attributes like `[ValidateSet()]`, `[ValidateRange()]`, etc., to constrain parameter input. Note: `[ValidateSet('Start','Stop')]` restricts choices.

6. **3.8** Prefer fully qualified .NET types for parameters (e.g., `[System.String]`) to prevent alias hijacking. Note: Use `[System.String]$Name` instead of `[string]$Name`.

7. **3.9** Support pipeline input with `[Parameter(ValueFromPipeline=$true)]` or `[Parameter(ValueFromPipelineByPropertyName=$true)]`. Note: Allows passing objects via the pipeline.

8. **13.3** Set `$ErrorActionPreference = 'Stop'` at script start and wrap code in `try/catch` blocks to handle exceptions explicitly. Note: Ensures all errors trigger catch logic.
