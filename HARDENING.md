<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-apply/v4** was hardened automatically. 23 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in action.yml and workflow files are pinned to mutable version tags or branch names instead of immutable 40-character SHA digests. This exposes the action to supply-chain attacks if any upstream action is compromised or its tag is moved.

Failing references in action.yml:
- actions/setup-node@v4
- actions/checkout@v4
- cloudposse/github-action-setup-atmos@v2
- cloudposse/github-action-atmos-get-setting@v2
- hashicorp/setup-terraform@v3
- cloudposse-github-actions/install-gh-releases@v1
- aws-actions/configure-aws-credentials@v4 (×3)
- cloudposse/github-action-terraform-plan-storage@v1 (×2)
- infracost/actions/setup@v3
- actions/cache@v4

Failing references in workflow files:
- actions/checkout@v4 (integration-tests.yml, test-atmos-pro-enabled.yml)
- cloudposse/github-action-atmos-terraform-plan@v4 (integration-tests.yml, test-atmos-pro-enabled.yml)
- cloudposse/.github/.github/workflows/shared-github-action.yml@main (branch.yml)
- cloudposse/.github/.github/workflows/shared-release-branches.yml@main (release.yml)
- cloudposse/github-actions-workflows-terraform-module/.github/workflows/scheduled.yml@main (scheduled.yml)
- nick-fields/assert-action@v2 (test-atmos-pro-enabled.yml)

Locations:

- `action.yml:63`
- `action.yml:69`
- `action.yml:76`
- `action.yml:82`
- `action.yml:155`
- `action.yml:161`
- `action.yml:170`
- `action.yml:214`
- `action.yml:248`
- `action.yml:270`
- `action.yml:296`
- `action.yml:316`
- `action.yml:336`
- `.github/workflows/branch.yml:19`
- `.github/workflows/integration-tests.yml:24`
- `.github/workflows/integration-tests.yml:46`
- `.github/workflows/integration-tests.yml:55`
- `.github/workflows/release.yml:9`
- `.github/workflows/scheduled.yml:10`
- `.github/workflows/test-atmos-pro-enabled.yml:26`
- `.github/workflows/test-atmos-pro-enabled.yml:52`
- `.github/workflows/test-atmos-pro-enabled.yml:61`
- `.github/workflows/test-atmos-pro-enabled.yml:76`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell commands (sub-rule a). This allows an attacker who controls the input values (via inputs.*, steps outputs, or github context) to inject arbitrary shell commands.

Violations:

1. 'Set atmos cli config path vars' (line ~74): `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — inputs.atmos-config-path interpolated directly into shell.

2. 'Define Job Control State Variables' (line ~178): `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV` — inputs.debug interpolated directly.

3. 'Check If GitHub Actions is Enabled For Component' (line ~184): `if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).github-actions-enabled }}" == "true" || "${{ fromJson(steps.atmos-settings.outputs.settings).atmos-pro-enabled }}" == "true" ]]` — step outputs interpolated directly into shell condition.

4. 'Set atmos cli base path vars' (line ~193): `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` — step output interpolated directly.

5. 'Define Job Variables' (line ~200): `STACK_NAME=$(echo "${{ inputs.stack }}" | sed ...)`, `COMPONENT_PATH=$( realpath ${{ fromJson(...).component-path }})`, `COMPONENT_NAME=$(echo "${{ inputs.component }}" | sed ...)`, `PLAN_FILE=".../$COMPONENT_SLUG-${{ inputs.sha }}.planfile"` — multiple inputs and step outputs interpolated directly.

6. 'Convert PLANFILE to JSON' (line ~298): `${{ fromJson(steps.atmos-settings.outputs.settings).command }} show -json "${{ steps.vars.outputs.plan_file }}"` — step output used as command name and argument.

7. 'Generate Infracost Diff' (line ~307): `--path="${{ steps.vars.outputs.plan_file }}.json"`, `--project-name "${{ inputs.stack }}-${{ inputs.component }}"` — inputs and step outputs interpolated directly.

8. 'Debug Infracost' (line ~318): `cat ${{ steps.vars.outputs.plan_file }}.json` — step output interpolated directly.

9. 'Terraform Apply' (line ~342): `-var "target:${{ inputs.stack }}-${{ inputs.component }}"`, `-var "job:${{ github.job }}"`, `--output "${{ github.workspace }}/atmos-apply-summary.md"`, `$([[ "${{ inputs.debug }}" == "true" ]] && ...)`, `apply -- atmos terraform apply ${{ inputs.component }} --stack ${{ inputs.stack }}` — multiple inputs and github context values interpolated directly.

In workflow files:
10. integration-tests.yml (line ~30): `mkdir -p ${{ runner.temp }}`, `cp ./tests/${{ matrix.platform }}/atmos.yaml ${{ runner.temp }}/atmos.yaml`, `sed -i -e 's#__STORAGE_REGION__#${{ env.AWS_REGION }}#g' ...` — runner, matrix, and env context values interpolated directly into shell.

11. test-atmos-pro-enabled.yml (line ~36): Same pattern with `${{ runner.temp }}` and `${{ env.AWS_REGION }}` interpolated directly into shell.

Locations:

- `action.yml:74`
- `action.yml:178`
- `action.yml:184`
- `action.yml:193`
- `action.yml:200`
- `action.yml:298`
- `action.yml:307`
- `action.yml:318`
- `action.yml:342`
- `.github/workflows/integration-tests.yml:30`
- `.github/workflows/test-atmos-pro-enabled.yml:36`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs and step outputs to $GITHUB_ENV and $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). This allows newline injection to set arbitrary environment variables or outputs.

Violations:

1. 'Set atmos cli config path vars' (~line 74): `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — inputs.atmos-config-path (unsanitized) written to GITHUB_ENV.

2. 'Define Job Control State Variables' (~line 178): `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV` — inputs.debug (unsanitized) written to GITHUB_ENV.

3. 'Set atmos cli base path vars' (~line 193): `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` then `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV` — step output (unsanitized) written to GITHUB_ENV.

4. 'Define Job Variables' (~line 200): Writes `inputs.stack`, `inputs.component`, `inputs.sha`, and step outputs to $GITHUB_OUTPUT without sanitization. E.g., `echo "stack_name=$STACK_NAME" >> $GITHUB_OUTPUT` where STACK_NAME is derived from `${{ inputs.stack }}`.

5. 'Set Infracost Variables' (~line 325): `echo "infracost_details_diff_breakdown=$INFRACOST_DETAILS_DIFF_BREAKDOWN" >> "$GITHUB_OUTPUT"` and `echo "infracost_diff_total_monthly_cost=$INFRACOST_DIFF_TOTAL_MONTHLY_COST" >> "$GITHUB_OUTPUT"` — values derived from step outputs written without sanitization.

Locations:

- `action.yml:74`
- `action.yml:178`
- `action.yml:193`
- `action.yml:200`
- `action.yml:325`

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

Fixed all security findings across action.yml and .github/workflows files:

1. unpinned-uses: Pinned all 10 action references in action.yml (setup-node, checkout, github-action-setup-atmos, github-action-atmos-get-setting, setup-terraform, install-gh-releases, configure-aws-credentials×3, github-action-terraform-plan-storage×2, infracost/actions/setup, actions/cache) and all workflow file references (branch.yml, release.yml, scheduled.yml, integration-tests.yml, test-atmos-pro-enabled.yml) to full 40-character SHA digests.

2. script-injection / static-inline-injection: Moved all ${{ inputs.* }}, ${{ github.* }}, ${{ steps.*.outputs.* }}, ${{ runner.* }}, ${{ matrix.* }}, and ${{ env.* }} expressions out of run: blocks into env: blocks. Shell scripts now reference plain environment variables. Used python3 to safely parse JSON settings output instead of inline fromJson() expressions.

3. github-env-injection: All values written to $GITHUB_ENV and $GITHUB_OUTPUT are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before being written to prevent newline injection attacks.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted shell expansion in the 'Terraform Apply' step of hardened/action/action.yml. Changed `&> ${TERRAFORM_OUTPUT_FILE}` to `&> "${TERRAFORM_OUTPUT_FILE}"` at offset ~23662. The TERRAFORM_OUTPUT_FILE variable is set from GH_RUN_ID which is sourced from `${{ github.run_id }}`, so it must be double-quoted per the security rules.

