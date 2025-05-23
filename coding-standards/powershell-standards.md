## ✅ Best Practices

- **Approved Verbs**  
  Use [approved verbs](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/approved-verbs-for-windows-powershell-commands) in function names.
  - Use approved verbs (Get-Verb) in Verb-Noun format (e.g., Get-UserAccount).

- **Descriptive Nouns**  
  Ensure function names clearly describe the action and the target.  
  [Cmdlet Guidelines](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/strongly-encouraged-development-guidelines)
  - Use specific, PascalCase nouns (e.g., Get-User for single, Get-Users for collections).

- **Comment-Based Help**  
  Include `.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, and `.EXAMPLE` sections.  
  [Help Guidelines](https://learn.microsoft.com/en-us/powershell/scripting/developer/help/writing-help-for-windows-powershell-scripts-and-functions)

- **Parameter Validation**  
  Use validation attributes like `[ValidateNotNullOrEmpty()]`, `[ValidateSet()]`, etc.

- **Error Handling**  
  Use `try/catch` and set `$ErrorActionPreference = 'Stop'` to catch exceptions explicitly.

- **Consistent Formatting**  
  Stick to consistent indentation (2 or 4 spaces) and logical sectioning.
  - Limit line length to 120 characters.

- **Avoid Aliases**  
  Do not use aliases like `?` or `ls`; prefer full cmdlet names for clarity and security.
  - Use full cmdlet names (e.g., Where-Object instead of `?`).

- **Use `[System.String]` Instead of `[string]`**  
  Prefer fully qualified .NET types to prevent alias hijacking and to align with PowerShell’s .NET foundation.

- **Use `#Requires` Statements**  
  Define environment prerequisites at the top of your script to ensure it fails fast when conditions aren't met.  
  Examples:
  ```powershell
  #Requires -PSEdition Desktop
  #Requires -Modules Az
  ```

## 📌 Additional Tips

- Use singular nouns for cmdlet names unless returning multiple items.
- Support pipeline input when applicable.

---

## 📚 References

- [Strongly Encouraged Cmdlet Guidelines](https://learn.microsoft.com/en-us/powershell/scripting/developer/cmdlet/strongly-encouraged-development-guidelines)
- [PowerShell Practice and Style Guide (PoshCode)](https://github.com/PoshCode/PowerShellPracticeAndStyle)
- [PowerShell Community Style Guide](https://learn.microsoft.com/en-us/powershell/scripting/community/contributing/powershell-style-guide)
