<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-apply/v4** was hardened automatically. 23 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 13 `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if any upstream action is compromised or a tag is moved. Failing references: actions/setup-node@v4, actions/checkout@v4, cloudposse/github-action-setup-atmos@v2, cloudposse/github-action-atmos-get-setting@v2, hashicorp/setup-terraform@v3, cloudposse-github-actions/install-gh-releases@v1, aws-actions/configure-aws-credentials@v4 (×3), cloudposse/github-action-terraform-plan-storage@v1 (×2), infracost/actions/setup@v3, actions/cache@v4.

Locations:

- `action.yml:63`
- `action.yml:67`
- `action.yml:73`
- `action.yml:80`
- `action.yml:143`
- `action.yml:148`
- `action.yml:158`
- `action.yml:207`
- `action.yml:232`
- `action.yml:249`
- `action.yml:265`
- `action.yml:280`
- `action.yml:320`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate GitHub Actions expressions into shell commands without routing through env vars, allowing an attacker-controlled value to break out of the shell context.

(a) 'Set atmos cli config path vars' step: `${{ inputs.atmos-config-path }}` is interpolated unquoted directly into a shell command: `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV`

(a) 'Define Job Control State Variables' step: `${{ inputs.debug }}` is interpolated directly: `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV`

(a) 'Check If GitHub Actions is Enabled For Component' step: `${{ fromJson(steps.atmos-settings.outputs.settings).github-actions-enabled }}` and `${{ fromJson(steps.atmos-settings.outputs.settings).atmos-pro-enabled }}` are interpolated directly into an `if [[ ... ]]` shell conditional.

(a) 'Set atmos cli base path vars' step: `${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}` is interpolated directly into a shell variable assignment.

(a) 'Define Job Variables' step: `${{ inputs.stack }}`, `${{ inputs.component }}`, `${{ inputs.sha }}`, and `${{ fromJson(steps.atmos-settings.outputs.settings).component-path }}` are all interpolated directly into shell commands.

(a) 'Convert PLANFILE to JSON' step (critical): `${{ fromJson(steps.atmos-settings.outputs.settings).command }}` is used as the first token of a shell command — an attacker controlling this value can execute arbitrary commands.

(a) 'Generate Infracost Diff' step: `${{ inputs.stack }}` and `${{ inputs.component }}` are interpolated directly into shell commands.

(a) 'Debug Infracost' step: `${{ steps.vars.outputs.plan_file }}` is interpolated unquoted.

(a) 'Set Infracost Variables' step: `${{ fromJson(steps.atmos-settings.outputs.settings).enable-infracost }}` is interpolated directly into an `if [[ ... ]]` conditional.

(a) 'Terraform Apply' step: `${{ inputs.stack }}`, `${{ inputs.component }}`, `${{ github.job }}`, `${{ inputs.debug }}`, `${{ inputs.branding-logo-image }}`, `${{ inputs.branding-logo-url }}`, `${{ github.repository }}`, and `${{ github.run_id }}` are all interpolated directly into shell commands.

Locations:

- `action.yml:71`
- `action.yml:168`
- `action.yml:174`
- `action.yml:183`
- `action.yml:189`
- `action.yml:285`
- `action.yml:291`
- `action.yml:302`
- `action.yml:307`
- `action.yml:333`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs directly to `$GITHUB_ENV` or `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`), allowing newline injection to set arbitrary environment variables or outputs.

'Set atmos cli config path vars' step: `${{ inputs.atmos-config-path }}` (caller-controlled) is written to `$GITHUB_ENV` via: `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV`

'Define Job Control State Variables' step: `${{ inputs.debug }}` (caller-controlled) is written to `$GITHUB_ENV` via: `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV`

'Set atmos cli base path vars' step: `${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}` (derived from step outputs, which are workflow-controllable) is written to `$GITHUB_ENV` via: `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV`

'Define Job Variables' step: Values derived from `${{ inputs.stack }}`, `${{ inputs.component }}`, `${{ inputs.sha }}`, and `${{ fromJson(steps.atmos-settings.outputs.settings).component-path }}` are written to `$GITHUB_OUTPUT` (stack_name, component_name, component_slug, component_path, cache-key, plan_file, lock_file) without sanitization.

Locations:

- `action.yml:71`
- `action.yml:168`
- `action.yml:184`
- `action.yml:199`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.atmos-config-path }}" appears directly in run: block of step "Set atmos cli config path vars"; move to env: map

Locations:

- `action.yml:74`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.debug }}" appears directly in run: block of step "Define Job Control State Variables"; move to env: map

Locations:

- `action.yml:210`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:236`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:238`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:241`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:351`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:351`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:356`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:356`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:406`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:406`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:407`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:408`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-image }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:412`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-url }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:413`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.debug }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:415`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:417`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:418`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:429`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:429`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection

**Notes:**

Rewrote action.yml with all security fixes:

1. **unpinned-uses**: Pinned all 13 `uses:` references to full 40-character SHA hashes with version tag comments (e.g., `actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4`).

2. **script-injection / static-inline-injection**: Moved all `${{ }}` expressions out of `run:` blocks into `env:` maps for every affected step. Steps fixed: 'Set atmos cli config path vars', 'Define Job Control State Variables', 'Check If GitHub Actions is Enabled For Component', 'Set atmos cli base path vars', 'Define Job Variables', 'Convert PLANFILE to JSON', 'Generate Infracost Diff', 'Debug Infracost', 'Set Infracost Variables', and 'Terraform Apply'. The 'Convert PLANFILE to JSON' step's critical command injection (where `${{ fromJson(...).command }}` was used as the first shell token) was fixed by placing it in an env var and quoting it: `"$ATMOS_COMMAND" show -json ...`.

3. **github-env-injection**: All values written to $GITHUB_ENV and $GITHUB_OUTPUT are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before writing, preventing newline injection. Applied to 'Set atmos cli config path vars' (ATMOS_CLI_CONFIG_PATH), 'Define Job Control State Variables' (DEBUG_ENABLED), 'Set atmos cli base path vars' (ATMOS_BASE_PATH), and 'Define Job Variables' (all 7 outputs).

### Iteration 1

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:
1. github-env-injection (line 77): Captured the output of `realpath "$safe_config_path"` into a new variable `safe_realpath` with `| tr -d '\n\r'` sanitization applied, then wrote `$safe_realpath` to $GITHUB_ENV instead of the unsanitized realpath output.
2. script-injection (line 530): Double-quoted the command substitution for `--log-level` so it reads `--log-level "$(...)"` instead of the unquoted `--log-level $(...)`, preventing word splitting of the expansion result.

