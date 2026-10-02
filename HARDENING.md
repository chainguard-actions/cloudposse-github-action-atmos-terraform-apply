<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-apply/v7.0.0** was hardened automatically. 41 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell command strings, violating rule (a). This allows an attacker-controlled value to be parsed as shell code before the shell ever sees it.

1. 'Set atmos cli config path vars': `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — inputs.atmos-config-path interpolated directly.
2. 'Define Job Control State Variables': `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV` — inputs.debug interpolated directly.
3. 'Check If GitHub Actions is Enabled For Component': `if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).github-actions-enabled }}" == "true" ...` — steps output interpolated directly.
4. 'Set atmos cli base path vars': `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` — steps output interpolated directly.
5. 'Define Job Variables': `STACK_NAME=$(echo "${{ inputs.stack }}" | sed ...)`, `COMPONENT_PATH=$( realpath ${{ fromJson(...).component-path }})`, `COMPONENT_NAME=$(echo "${{ inputs.component }}" | sed ...)`, `...-${{ inputs.sha }}.planfile` — multiple inputs interpolated directly.
6. 'Plan prepare': `if [[ -n "${{ inputs.identity }}" ]]`, `base_cmd+=" --identity=${{ inputs.identity }}"`, `-var "target:${{ inputs.stack }}-${{ inputs.component }}"`, `-var "stack:${{ inputs.stack }}"`, `-var "job:${{ github.job }}"`, `atmos terraform plan ${{ inputs.component }}`, `--stack ${{ inputs.stack }}`, `-out=${{ steps.vars.outputs.renewed_plan_file }}`, `if [[ "${{ inputs.plan-storage }}" == "true" ]]`, `atmos terraform plan-diff ${{ inputs.component }}`, `--stack ${{ inputs.stack }}` — many inputs and github context values interpolated directly.
7. 'Determine Plan File': `if [[ "${{ inputs.skip-plandiff }}" == "true" ]]`, `echo "plan_file=${{ steps.vars.outputs.retrieved_plan_file }}"`, `echo "plan_filename=${{ steps.vars.outputs.retrieved_plan_filename }}"` — inputs and steps outputs interpolated directly.
8. 'Check Whether Infracost is Enabled': `if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).enable-infracost }}" == "true" ]]` — steps output interpolated directly.
9. 'Set Infracost Variables': `if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).enable-infracost }}" == "true" ]]` — steps output interpolated directly.
10. 'Terraform Apply': `if [[ "${{ inputs.skip-plandiff }}" != "true" ]]`, `-var "target:${{ inputs.stack }}-${{ inputs.component }}"`, `-var "stack:${{ inputs.stack }}"`, `-var "job:${{ github.job }}"`, `atmos terraform deploy ${{ inputs.component }}`, `--stack ${{ inputs.stack }}`, `--planfile ${{ steps.plan-file.outputs.plan_filename }}`, `atmos terraform output ${{ inputs.component }}`, `--stack ${{ inputs.stack }}`, `1> ${{ steps.vars.outputs.component_path }}/output_values.json`, `cd ${{ steps.vars.outputs.component_path }}` — many inputs and steps outputs interpolated directly.

All ${{ }} expressions must be moved to env: variables and those env vars must be double-quoted in the shell script.

Locations:

- `action.yml:74`
- `action.yml:233`
- `action.yml:240`
- `action.yml:251`
- `action.yml:262`
- `action.yml:430`
- `action.yml:500`
- `action.yml:515`
- `action.yml:530`
- `action.yml:580`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs to $GITHUB_ENV or $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. 'Set atmos cli config path vars': `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — inputs.atmos-config-path (caller-controlled) written to GITHUB_ENV unsanitized.
2. 'Define Job Control State Variables': `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV` — inputs.debug written to GITHUB_ENV unsanitized.
3. 'Define Job Variables': STACK_NAME, COMPONENT_NAME, and RETRIEVED_PLAN_FILENAME are derived from inputs.stack, inputs.component, and inputs.sha respectively, then written to $GITHUB_OUTPUT without sanitization (e.g. `echo "stack_name=$STACK_NAME" >> $GITHUB_OUTPUT`).
4. 'Determine Plan File': `echo "plan_file=${{ steps.vars.outputs.retrieved_plan_file }}" >> $GITHUB_OUTPUT` and `echo "plan_filename=${{ steps.vars.outputs.retrieved_plan_filename }}" >> $GITHUB_OUTPUT` — steps outputs (derived from untrusted inputs) written to GITHUB_OUTPUT unsanitized.

An attacker can inject newlines into these values to set arbitrary environment variables or outputs.

Locations:

- `action.yml:74`
- `action.yml:233`
- `action.yml:262`
- `action.yml:500`

### unpinned-uses (severity: high)

All 13 uses: references in action.yml use mutable version tags instead of pinned 40-character SHA digests. This exposes the action to supply-chain attacks where a compromised or malicious tag update could execute arbitrary code in the runner.

Failing references:
- uses: actions/setup-node@v4
- uses: actions/checkout@v4
- uses: cloudposse/github-action-setup-atmos@v2
- uses: cloudposse/github-action-atmos-get-setting@v2
- uses: hashicorp/setup-terraform@v3
- uses: cloudposse-github-actions/install-gh-releases@v1
- uses: aws-actions/configure-aws-credentials@v4 (appears multiple times)
- uses: cloudposse/github-action-terraform-plan-storage@v1 (appears multiple times)
- uses: actions/cache@v4
- uses: infracost/actions/setup@v3

All should be pinned to their full 40-character commit SHA, e.g. `uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:66`
- `action.yml:70`
- `action.yml:79`
- `action.yml:86`
- `action.yml:196`
- `action.yml:203`
- `action.yml:215`
- `action.yml:300`
- `action.yml:340`
- `action.yml:380`
- `action.yml:415`
- `action.yml:420`
- `action.yml:530`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.atmos-config-path }}" appears directly in run: block of step "Set atmos cli config path vars"; move to env: map

Locations:

- `action.yml:82`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.debug }}" appears directly in run: block of step "Define Job Control State Variables"; move to env: map

Locations:

- `action.yml:219`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:245`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:247`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha }}" appears directly in run: block of step "Define Job Variables"; move to env: map

Locations:

- `action.yml:252`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.identity }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:379`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.identity }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:380`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:386`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:386`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:387`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:388`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-image }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:390`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-url }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:391`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.debug }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:393`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:395`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:396`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.plan-storage }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:415`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:417`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:418`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:435`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Plan prepare"; move to env: map

Locations:

- `action.yml:437`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.skip-plandiff }}" appears directly in run: block of step "Determine Plan File"; move to env: map

Locations:

- `action.yml:449`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:490`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:490`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:495`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Generate Infracost Diff"; move to env: map

Locations:

- `action.yml:495`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.skip-plandiff }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:538`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:544`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:544`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:545`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:546`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-image }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:550`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.branding-logo-url }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:551`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.debug }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:553`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:555`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:556`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.component }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:569`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.stack }}" appears directly in run: block of step "Terraform Apply"; move to env: map

Locations:

- `action.yml:570`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection, github-env-injection

**Notes:**

All 13 uses: references pinned to full SHA digests. All ${{ }} expressions moved from run: blocks to env: blocks across all affected steps (Set atmos cli config path vars, Define Job Control State Variables, Check If GitHub Actions is Enabled For Component, Set atmos cli base path vars, Define Job Variables, Plan prepare, Determine Plan File, Check Whether Infracost is Enabled, Set Infracost Variables, Convert PLANFILE to JSON, Debug Infracost, Generate Infracost Diff, Terraform Apply). Values written to $GITHUB_ENV and $GITHUB_OUTPUT sanitized with printf '%s' | tr -d '\n\r' to prevent newline injection attacks.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed script-injection in both 'Plan prepare' and 'Terraform Apply' steps by converting base_cmd from a string to a bash array (base_cmd=() and base_cmd+=("--identity=$INPUT_IDENTITY")), then expanding with "${base_cmd[@]}" to prevent word-splitting and command injection from the attacker-controllable INPUT_IDENTITY value. Fixed github-env-injection in 'Set atmos cli base path vars' by adding tr -d '\n\r' sanitization before writing ATMOS_BASE_PATH to $GITHUB_ENV. Fixed github-env-injection in 'Define Job Variables' by sanitizing COMPONENT_PATH and COMPONENT_CACHE_KEY (both derived from SETTINGS_COMPONENT_PATH) with printf '%s' ... | tr -d '\n\r' before they are used to construct values written to $GITHUB_OUTPUT.

