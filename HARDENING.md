<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-apply/v4** was hardened automatically. 23 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if any upstream action is compromised or a tag is moved. Failing references: actions/setup-node@v4, actions/checkout@v4, cloudposse/github-action-setup-atmos@v2, cloudposse/github-action-atmos-get-setting@v2, hashicorp/setup-terraform@v3, cloudposse-github-actions/install-gh-releases@v1, aws-actions/configure-aws-credentials@v4 (×3), cloudposse/github-action-terraform-plan-storage@v1 (×2), infracost/actions/setup@v3, actions/cache@v4.

Locations:

- `action.yml:63`
- `action.yml:67`
- `action.yml:72`
- `action.yml:100`
- `action.yml:148`
- `action.yml:155`
- `action.yml:168`
- `action.yml:219`
- `action.yml:261`
- `action.yml:285`
- `action.yml:302`
- `action.yml:330`
- `action.yml:360`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate GitHub Actions expressions (${{ ... }}) inside shell command strings (rule a), allowing an attacker who controls input values to inject arbitrary shell commands. Affected steps and offending expressions:

1. 'Set atmos cli config path vars' (~line 70): `$(realpath ${{ inputs.atmos-config-path }})` — inputs.atmos-config-path interpolated unquoted into shell.

2. 'Define Job Control State Variables' (~line 183): `echo "DEBUG_ENABLED=${{ inputs.debug }}"` — inputs.debug interpolated directly.

3. 'Check If GitHub Actions is Enabled For Component' (~line 190): `if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).github-actions-enabled }}" == "true" ...` — step output interpolated in shell condition.

4. 'Set atmos cli base path vars' (~line 200): `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` — step output interpolated in shell.

5. 'Define Job Variables' (~line 208): `STACK_NAME=$(echo "${{ inputs.stack }}" ...)`, `COMPONENT_PATH=$( realpath ${{ fromJson(...).component-path }})`, `COMPONENT_NAME=$(echo "${{ inputs.component }}" ...)`, `PLAN_FILE=".../$COMPONENT_SLUG-${{ inputs.sha }}.planfile"` — multiple inputs and step outputs interpolated directly.

6. 'Convert PLANFILE to JSON' (~line 280): `${{ fromJson(steps.atmos-settings.outputs.settings).command }} show -json "${{ steps.vars.outputs.plan_file }}"` — step outputs used as command name and argument.

7. 'Generate Infracost Diff' (~line 291): `--path="${{ steps.vars.outputs.plan_file }}.json"`, `--project-name "${{ inputs.stack }}-${{ inputs.component }}"` — inputs and step outputs interpolated.

8. 'Debug Infracost' (~line 310): `cat ${{ steps.vars.outputs.plan_file }}.json` — step output interpolated unquoted.

9. 'Set Infracost Variables' (~line 317): `if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).enable-infracost }}" == "true" ]]` — step output interpolated in condition.

10. 'Terraform Apply' (~line 340): `atmos terraform apply ${{ inputs.component }}`, `--stack ${{ inputs.stack }}`, `-var "job:${{ github.job }}"`, `--log-level $([[ "${{ inputs.debug }}" == "true" ]] ...)`, `--output "${{ github.workspace }}/..."`, `atmos terraform output ${{ inputs.component }} --stack ${{ inputs.stack }}`, `sed -i ... ${{ github.workspace }}/...`, `echo "[Job](https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }})"` — numerous inputs and github context values interpolated directly.

Locations:

- `action.yml:70`
- `action.yml:183`
- `action.yml:190`
- `action.yml:200`
- `action.yml:208`
- `action.yml:280`
- `action.yml:291`
- `action.yml:310`
- `action.yml:317`
- `action.yml:340`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs or step outputs to $GITHUB_ENV or $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). This allows an attacker to inject arbitrary environment variables or output values by embedding newlines in the input.

1. 'Set atmos cli config path vars' (~line 70): `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — inputs.atmos-config-path written to GITHUB_ENV unsanitized.

2. 'Define Job Control State Variables' (~line 183): `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV` — inputs.debug written to GITHUB_ENV unsanitized.

3. 'Set atmos cli base path vars' (~line 200): `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV` where ATMOS_BASE_PATH is set from `${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}` — step output written to GITHUB_ENV unsanitized.

4. 'Define Job Variables' (~line 208): `echo "stack_name=$STACK_NAME" >> $GITHUB_OUTPUT`, `echo "component_name=$COMPONENT_NAME" >> $GITHUB_OUTPUT`, `echo "plan_file=$PLAN_FILE" >> $GITHUB_OUTPUT`, etc. — variables derived from inputs.stack, inputs.component, inputs.sha, and step outputs written to GITHUB_OUTPUT unsanitized.

Locations:

- `action.yml:70`
- `action.yml:183`
- `action.yml:200`
- `action.yml:208`

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

Rewrote action.yml to fix all security findings:

1. **unpinned-uses**: Pinned all 13 `uses:` references to full 40-char SHA digests with tag comments (setup-node, checkout, setup-atmos, atmos-get-setting, setup-terraform, install-gh-releases, configure-aws-credentials ×3, terraform-plan-storage ×2, infracost/actions/setup, actions/cache).

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions out of `run:` blocks into `env:` blocks for every affected step: 'Set atmos cli config path vars', 'Define Job Control State Variables', 'Check If GitHub Actions is Enabled For Component', 'Set atmos cli base path vars', 'Define Job Variables', 'Convert PLANFILE to JSON', 'Generate Infracost Diff', 'Debug Infracost', 'Set Infracost Variables', 'Check Whether Infracost is Enabled', and 'Terraform Apply'. Shell scripts now reference plain environment variables ($VAR_NAME).

3. **github-env-injection**: All values written to $GITHUB_ENV or $GITHUB_OUTPUT that derive from inputs or step outputs are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before writing, preventing newline injection. This covers: ATMOS_CLI_CONFIG_PATH, DEBUG_ENABLED, ATMOS_BASE_PATH, and all outputs in 'Define Job Variables'.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:
1. github-env-injection (line 214): Added `safe=$(printf '%s' "$safe" | tr -d '\n\r')` after the realpath call in the 'Set atmos cli base path vars' step to strip newlines before writing to $GITHUB_ENV.
2. script-injection (line 395): Replaced the unquoted command substitution `$([[ "$INPUT_DEBUG" == "true" ]] && echo "DEBUG" || echo "INFO")` with a pre-computed variable `LOG_LEVEL` (set via an if/then block) that is then passed as `"$LOG_LEVEL"` (double-quoted) to the --log-level argument of tfcmt.

