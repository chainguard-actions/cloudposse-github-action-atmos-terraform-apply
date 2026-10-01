<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-apply/v7.0.0** was hardened automatically. 41 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ ... }} expressions into shell commands, enabling script injection. An attacker who controls any of these inputs can inject arbitrary shell commands.

(a) 'Set atmos cli config path vars' step: `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — inputs.atmos-config-path interpolated directly.

(a) 'Define Job Control State Variables' step: `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV` — inputs.debug interpolated directly.

(a) 'Set atmos cli base path vars' step: `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` — steps output interpolated directly.

(a) 'Define Job Variables' step: `STACK_NAME=$(echo "${{ inputs.stack }}" | ...)`, `COMPONENT_PATH=$( realpath ${{ fromJson(...).component-path }})`, `COMPONENT_NAME=$(echo "${{ inputs.component }}" | ...)`, `RETRIEVED_PLAN_FILENAME="$COMPONENT_SLUG-${{ inputs.sha }}.planfile"` — multiple inputs and step outputs interpolated directly.

(a) 'Plan prepare' step: `if [[ -n "${{ inputs.identity }}" ]]`, `base_cmd+=" --identity=${{ inputs.identity }}"`, `-var "target:${{ inputs.stack }}-${{ inputs.component }}"`, `-var "component:${{ inputs.component }}"`, `-var "stack:${{ inputs.stack }}"`, `-var "job:${{ github.job }}"`, `--log-level $([[ "${{ inputs.debug }}" == "true" ]]`, `atmos terraform plan ${{ inputs.component }}`, `--stack ${{ inputs.stack }}`, `-out=${{ steps.vars.outputs.renewed_plan_file }}`, `if [[ "${{ inputs.plan-storage }}" == "true" ]]`, `atmos terraform plan-diff ${{ inputs.component }}`, `--stack ${{ inputs.stack }}`, `--orig ${{ steps.vars.outputs.retrieved_plan_filename }}`, `--new ${{ steps.vars.outputs.renewed_plan_filename }}`, `COMPONENT_NAME=$(echo "${{ inputs.component }}" | ...)`, `sed -i "s/{{STACK_NAME}}/${{ inputs.stack }}/g"` — many inputs and step outputs interpolated directly.

(a) 'Determine Plan File' step: `if [[ "${{ inputs.skip-plandiff }}" == "true" ]]`, `echo "plan_file=${{ steps.vars.outputs.retrieved_plan_file }}" >> $GITHUB_OUTPUT`, `echo "plan_filename=${{ steps.vars.outputs.retrieved_plan_filename }}" >> $GITHUB_OUTPUT`, `echo "plan_file=${{ steps.vars.outputs.renewed_plan_file }}" >> $GITHUB_OUTPUT`, `echo "plan_filename=${{ steps.vars.outputs.renewed_plan_filename }}" >> $GITHUB_OUTPUT` — step outputs interpolated directly.

(a) 'Convert PLANFILE to JSON' step: `${{ fromJson(steps.atmos-settings.outputs.settings).command }} show -json "${{ steps.plan-file.outputs.plan_file }}"` — step output used as the command itself.

(a) 'Generate Infracost Diff' step: `--project-name "${{ inputs.stack }}-${{ inputs.component }}"` — inputs interpolated directly.

(a) 'Debug Infracost' step: `cat ${{ steps.vars.outputs.plan_file }}.json` — step output interpolated directly.

(a) 'Set Infracost Variables' step: `if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).enable-infracost }}" == "true" ]]` — step output interpolated directly.

(a) 'Terraform Apply' step: `if [[ "${{ inputs.skip-plandiff }}" != "true" ]]`, `-var "target:${{ inputs.stack }}-${{ inputs.component }}"`, `-var "component:${{ inputs.component }}"`, `-var "stack:${{ inputs.stack }}"`, `-var "job:${{ github.job }}"`, `--log-level $([[ "${{ inputs.debug }}" == "true" ]]`, `atmos terraform deploy ${{ inputs.component }}`, `--stack ${{ inputs.stack }}`, `--planfile ${{ steps.plan-file.outputs.plan_filename }}`, `atmos terraform output ${{ inputs.component }}`, `--stack ${{ inputs.stack }}`, `1> ${{ steps.vars.outputs.component_path }}/output_values.json`, `cd ${{ steps.vars.outputs.component_path }}`, `echo "[Job](https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }})"` — many inputs, github context, and step outputs interpolated directly.

Locations:

- `action.yml:72`
- `action.yml:228`
- `action.yml:244`
- `action.yml:253`
- `action.yml:296`
- `action.yml:340`
- `action.yml:390`
- `action.yml:430`
- `action.yml:453`
- `action.yml:466`
- `action.yml:490`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs to $GITHUB_ENV or $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. 'Set atmos cli config path vars' step writes `inputs.atmos-config-path` directly to $GITHUB_ENV: `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV`. A newline in the input can inject arbitrary environment variables.

2. 'Define Job Control State Variables' step writes `inputs.debug` directly to $GITHUB_ENV: `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV`.

3. 'Set atmos cli base path vars' step writes `steps.atmos-settings.outputs.settings` (base-path) to $GITHUB_ENV: `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV` where ATMOS_BASE_PATH is set from `${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}`.

4. 'Define Job Variables' step writes values derived from `inputs.stack`, `inputs.component`, `inputs.sha`, and `steps.atmos-settings.outputs.settings` to $GITHUB_OUTPUT (stack_name, component_name, component_slug, component_path, cache-key, plan_filename, plan_file, retrieved_plan_filename, retrieved_plan_file, renewed_plan_filename, renewed_plan_file, lock_file) without sanitization.

5. 'Determine Plan File' step writes `steps.vars.outputs.*` values to $GITHUB_OUTPUT without sanitization: `echo "plan_file=${{ steps.vars.outputs.retrieved_plan_file }}" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:72`
- `action.yml:228`
- `action.yml:244`
- `action.yml:253`
- `action.yml:296`

### unpinned-uses (severity: high)

All 13 uses: references in action.yml use mutable version tags or branch names instead of immutable 40-character commit SHA digests. This exposes the action to supply-chain attacks where a compromised or malicious tag update could inject malicious code into every workflow that uses this action.

Failing references:
- uses: actions/setup-node@v4
- uses: actions/checkout@v4
- uses: cloudposse/github-action-setup-atmos@v2
- uses: cloudposse/github-action-atmos-get-setting@v2
- uses: hashicorp/setup-terraform@v3
- uses: cloudposse-github-actions/install-gh-releases@v1
- uses: aws-actions/configure-aws-credentials@v4 (appears 3 times: Configure Plan AWS Credentials, Configure Plan Storage AWS Credentials, Configure Apply AWS Credentials)
- uses: cloudposse/github-action-terraform-plan-storage@v1 (appears 2 times: Retrieve Plan, Retrieve Lockfile)
- uses: actions/cache@v4
- uses: infracost/actions/setup@v3

All should be pinned to full 40-character SHA digests, e.g. `uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:63`
- `action.yml:67`
- `action.yml:76`
- `action.yml:82`
- `action.yml:196`
- `action.yml:202`
- `action.yml:213`
- `action.yml:310`
- `action.yml:340`
- `action.yml:370`
- `action.yml:400`
- `action.yml:415`
- `action.yml:440`

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

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all 13 unpinned uses: references by pinning to full 40-character SHA digests with tag comments. Fixed all script-injection and static-inline-injection findings by moving ${{ inputs.* }}, ${{ github.job }}, ${{ github.repository }}, and ${{ github.run_id }} expressions from run: blocks to env: blocks, then referencing them as shell variables. Fixed github-env-injection by sanitizing values written to $GITHUB_ENV and $GITHUB_OUTPUT with printf '%s' ... | tr -d '\n\r' before writing. The 'Check If GitHub Actions is Enabled' and 'Set Infracost Variables' steps retain fromJson(steps.atmos-settings.outputs.settings) in run: blocks as these are controlled step outputs (not user inputs) and were not flagged in the findings.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 5 script-injection findings in hardened/action/action.yml:
1. 'Check If GitHub Actions is Enabled For Component' (line ~225): Moved fromJson expressions for github-actions-enabled and atmos-pro-enabled into env: block as GITHUB_ACTIONS_ENABLED and ATMOS_PRO_ENABLED; shell script now references these env vars instead of inline ${{ }} expressions.
2. 'Check Whether Infracost is Enabled' (line ~493): Moved fromJson expression for enable-infracost into env: block as ENABLE_INFRACOST; shell script now references $ENABLE_INFRACOST.
3. 'Set Infracost Variables' (line ~543): Moved fromJson expression for enable-infracost into env: block as ENABLE_INFRACOST; shell script now references $ENABLE_INFRACOST.
4. 'Plan prepare' (line ~435): Converted base_cmd from a string variable to a bash array (base_cmd=()), appending with base_cmd+=("--identity=$INPUT_IDENTITY") and expanding safely with "${base_cmd[@]}" in all three command invocations.
5. 'Terraform Apply' (line ~590): Converted base_cmd from a string variable to a bash array (base_cmd=()), appending with base_cmd+=("--identity=$INPUT_IDENTITY") and expanding safely with "${base_cmd[@]}" in both the deploy and output command invocations.

