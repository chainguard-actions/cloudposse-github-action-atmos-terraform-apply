<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-apply/v4** was hardened automatically. 23 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks. Failing references: actions/setup-node@v4, actions/checkout@v4, cloudposse/github-action-setup-atmos@v2, cloudposse/github-action-atmos-get-setting@v2, hashicorp/setup-terraform@v3, cloudposse-github-actions/install-gh-releases@v1, aws-actions/configure-aws-credentials@v4 (used 3 times), cloudposse/github-action-terraform-plan-storage@v1 (used 2 times), infracost/actions/setup@v3, actions/cache@v4.

Locations:

- `action.yml:63`
- `action.yml:67`
- `action.yml:72`
- `action.yml:83`
- `action.yml:130`
- `action.yml:137`
- `action.yml:148`
- `action.yml:176`
- `action.yml:196`
- `action.yml:210`
- `action.yml:230`
- `action.yml:248`
- `action.yml:270`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions — including inputs.*, github.*, and steps.*.outputs.* — inside shell command strings (sub-rule a). YAML template substitution occurs before the shell parses the string, so an attacker-controlled value can inject arbitrary shell commands. Violations: (1) 'Set atmos cli config path vars': echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV. (2) 'Define Job Control State Variables': echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV. (3) 'Check If GitHub Actions is Enabled For Component': if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).github-actions-enabled }}" == "true" ]]. (4) 'Set atmos cli base path vars': ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}". (5) 'Define Job Variables': STACK_NAME=$(echo "${{ inputs.stack }}" | sed ...), COMPONENT_PATH=$( realpath ${{ fromJson(...).component-path }}), COMPONENT_NAME=$(echo "${{ inputs.component }}" | sed ...), PLAN_FILE=".../$COMPONENT_SLUG-${{ inputs.sha }}.planfile". (6) 'Check Whether Infracost is Enabled': if [[ "${{ fromJson(...).enable-infracost }}" == "true" ]]. (7) 'Convert PLANFILE to JSON': ${{ fromJson(...).command }} show -json "${{ steps.vars.outputs.plan_file }}" — step output used as command prefix. (8) 'Generate Infracost Diff': --path="${{ steps.vars.outputs.plan_file }}.json", --project-name "${{ inputs.stack }}-${{ inputs.component }}". (9) 'Debug Infracost': cat ${{ steps.vars.outputs.plan_file }}.json. (10) 'Set Infracost Variables': if [[ "${{ fromJson(...).enable-infracost }}" == "true" ]]. (11) 'Terraform Apply': atmos terraform apply ${{ inputs.component }} --stack ${{ inputs.stack }}, -var "job:${{ github.job }}", --log-level $([[ "${{ inputs.debug }}" == "true" ]] && ...), atmos terraform output ${{ inputs.component }} --stack ${{ inputs.stack }}, echo "[Job](https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }})" >> atmos-apply-summary.md.

Locations:

- `action.yml:68`
- `action.yml:155`
- `action.yml:163`
- `action.yml:172`
- `action.yml:180`
- `action.yml:200`
- `action.yml:220`
- `action.yml:228`
- `action.yml:238`
- `action.yml:244`
- `action.yml:270`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs or step outputs to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). An attacker can inject newlines to add arbitrary environment variable definitions affecting subsequent steps. Violations: (1) 'Set atmos cli config path vars': echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV — inputs.atmos-config-path written to GITHUB_ENV without sanitization. (2) 'Define Job Control State Variables': echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV — inputs.debug written to GITHUB_ENV without sanitization. (3) 'Set atmos cli base path vars': ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}" then echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV — a workflow-controllable step output written to GITHUB_ENV without sanitization.

Locations:

- `action.yml:68`
- `action.yml:155`
- `action.yml:172`

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

Fixed all security findings in hardened/action/action.yml:

1. **unpinned-uses**: Pinned all 13 `uses:` references to full 40-character SHA digests with version tag comments.

2. **script-injection / static-inline-injection**: Moved all `${{ inputs.* }}`, `${{ github.job }}`, `${{ github.repository }}`, `${{ github.run_id }}`, `${{ steps.*.outputs.* }}`, and `${{ fromJson(...) }}` expressions from `run:` blocks into `env:` maps. Shell scripts now reference these as plain environment variables.

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing user-controlled values (inputs.atmos-config-path, inputs.debug, and the base-path step output) to $GITHUB_ENV.

The remaining `${{ github.workspace }}` references in the Terraform Apply `run:` block are safe (GitHub-controlled path, not user-controlled input). The `${{ }}` expressions in `with:`, `if:`, and action `settings:` YAML blocks are also safe as they are not shell-interpolated.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:

1. script-injection: Replaced all 7 occurrences of `${{ github.workspace }}` in the 'Terraform Apply' run block with `$GITHUB_WORKSPACE` (the standard pre-set environment variable). This eliminates inline expression interpolation in shell commands.

2. github-env-injection: Added sanitization to the 'Define Job Variables' step. All 7 values written to $GITHUB_OUTPUT (stack_name, component_name, component_slug, component_path, cache-key, plan_file, lock_file) are now passed through `printf '%s' "$VAR" | tr -d '\n\r'` before being written, preventing newline injection from attacker-controlled inputs (inputs.stack, inputs.component, inputs.sha).

