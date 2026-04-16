# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `c40cfe5fa14e08549b1b988e7e5a26da4816abf0`

**Test Policy SHA:** `f2e7d85641cde4267138117189b8eba7ba2bfbde`

Action **cloudposse--github-action-atmos-terraform-apply/v4** was hardened automatically. 23 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 13 'uses:' references in action.yml use mutable version tags instead of pinned 40-character commit SHAs. This exposes the action to supply-chain attacks where a compromised upstream action tag could execute arbitrary code. Failing references:
- uses: actions/setup-node@v4 (line 63)
- uses: actions/checkout@v4 (line 69)
- uses: cloudposse/github-action-setup-atmos@v2 (line 77)
- uses: cloudposse/github-action-atmos-get-setting@v2 (line 84)
- uses: hashicorp/setup-terraform@v3
- uses: cloudposse-github-actions/install-gh-releases@v1
- uses: aws-actions/configure-aws-credentials@v4 (three occurrences)
- uses: cloudposse/github-action-terraform-plan-storage@v1 (two occurrences)
- uses: infracost/actions/setup@v3
- uses: actions/cache@v4

Locations:

- `action.yml:63`
- `action.yml:69`
- `action.yml:77`
- `action.yml:84`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ inputs.* }} and ${{ github.* }} expressions into shell commands without first assigning them to environment variables. An attacker who controls these inputs (e.g. via workflow_dispatch or a crafted pull request) can inject arbitrary shell commands.

1. 'Set atmos cli config path vars' step (line ~74): `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — inputs.atmos-config-path is interpolated directly into a shell subcommand.

2. 'Define Job Control State Variables' step (line ~216): `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV` — inputs.debug is interpolated directly.

3. 'Define Job Variables' step (line ~240): `STACK_NAME=$(echo "${{ inputs.stack }}" | sed ...)`, `PLAN_FILE=".../$COMPONENT_SLUG-${{ inputs.sha }}.planfile"`, `COMPONENT_NAME=$(echo "${{ inputs.component }}" | sed ...)` — inputs.stack, inputs.component, and inputs.sha are all interpolated directly.

4. 'Terraform Apply' step (line ~424): `atmos terraform apply ${{ inputs.component }} --stack ${{ inputs.stack }}`, `-var "target:${{ inputs.stack }}-${{ inputs.component }}"`, `-var "job:${{ github.job }}"`, `$([[ "${{ inputs.debug }}" == "true" ]] && ...)` — inputs.component, inputs.stack, inputs.debug, and github.job are all interpolated directly into shell commands.

Locations:

- `action.yml:74`
- `action.yml:216`
- `action.yml:240`
- `action.yml:424`

### github-env-injection (severity: high)

Multiple run: blocks write attacker-controlled inputs.* values to $GITHUB_ENV or $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). An attacker can inject newlines into these values to set arbitrary environment variables or outputs, potentially escalating privileges or hijacking subsequent steps.

1. 'Set atmos cli config path vars' step (line ~74): `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — inputs.atmos-config-path is written to $GITHUB_ENV without sanitization.

2. 'Define Job Control State Variables' step (line ~216): `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV` — inputs.debug is written to $GITHUB_ENV without sanitization.

3. 'Define Job Variables' step (line ~240): STACK_NAME and COMPONENT_NAME are derived from ${{ inputs.stack }} and ${{ inputs.component }} respectively, and then written to $GITHUB_OUTPUT (e.g. `echo "stack_name=$STACK_NAME" >> $GITHUB_OUTPUT`) without sanitization. Similarly, PLAN_FILE incorporates ${{ inputs.sha }} and is written to $GITHUB_OUTPUT.

Locations:

- `action.yml:74`
- `action.yml:216`
- `action.yml:240`

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

Fixed all 23 findings in action.yml:

1. unpinned-uses: Pinned all 13 uses: references to full 40-char commit SHAs (actions/setup-node, actions/checkout, cloudposse/github-action-setup-atmos, cloudposse/github-action-atmos-get-setting, hashicorp/setup-terraform, cloudposse-github-actions/install-gh-releases, aws-actions/configure-aws-credentials x3, cloudposse/github-action-terraform-plan-storage x2, infracost/actions/setup, actions/cache).

2. script-injection / static-inline-injection: Moved all ${{ inputs.* }} and ${{ github.job }} expressions from run: shell blocks into env: maps for the following steps: 'Set atmos cli config path vars', 'Define Job Control State Variables', 'Define Job Variables', 'Generate Infracost Diff', and 'Terraform Apply'. Shell scripts now reference plain environment variables ($INPUT_STACK, $INPUT_COMPONENT, $INPUT_SHA, $INPUT_DEBUG, $INPUT_BRANDING_LOGO_IMAGE, $INPUT_BRANDING_LOGO_URL, $GITHUB_JOB_NAME, $ATMOS_CONFIG_PATH_INPUT).

3. github-env-injection: All user-controlled values written to $GITHUB_ENV or $GITHUB_OUTPUT are sanitized with printf '%s' "$VAR" | tr -d '\n\r' before use to prevent newline injection attacks.

