<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **cloudposse--github-action-atmos-terraform-apply/v4** was hardened automatically. 23 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks. Failing references include: `actions/setup-node@v4`, `actions/checkout@v4`, `cloudposse/github-action-setup-atmos@v2`, `cloudposse/github-action-atmos-get-setting@v2`, `hashicorp/setup-terraform@v3`, `cloudposse-github-actions/install-gh-releases@v1`, `aws-actions/configure-aws-credentials@v4` (x3), `cloudposse/github-action-terraform-plan-storage@v1` (x2), `infracost/actions/setup@v3`, `actions/cache@v4`.

Locations:

- `action.yml:57`
- `action.yml:61`
- `action.yml:68`
- `action.yml:73`
- `action.yml:136`
- `action.yml:141`
- `action.yml:148`
- `action.yml:185`
- `action.yml:200`
- `action.yml:210`
- `action.yml:222`
- `action.yml:253`
- `action.yml:291`

### script-injection (severity: high)

Multiple run: blocks directly interpolate GitHub Actions expressions (${{ ... }}) inside shell commands, enabling script injection (sub-rule a). Violations: (1) 'Set atmos cli config path vars': `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})"` — attacker-controlled input in shell. (2) 'Define Job Control State Variables': `echo "DEBUG_ENABLED=${{ inputs.debug }}"`. (3) 'Check If GitHub Actions is Enabled': `if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).github-actions-enabled }}" == "true" ...`. (4) 'Set atmos cli base path vars': `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"`. (5) 'Define Job Variables': `STACK_NAME=$(echo "${{ inputs.stack }}")`, `COMPONENT_PATH=$( realpath ${{ fromJson(...).component-path }})`, `PLAN_FILE=".../$COMPONENT_SLUG-${{ inputs.sha }}.planfile"`. (6) 'Check Whether Infracost is Enabled': `if [[ "${{ fromJson(...).enable-infracost }}" == "true" ]]`. (7) 'Convert PLANFILE to JSON': `${{ fromJson(...).command }} show -json "${{ steps.vars.outputs.plan_file }}"` — step output used as command name. (8) 'Generate Infracost Diff': `--path="${{ steps.vars.outputs.plan_file }}.json"`, `--project-name "${{ inputs.stack }}-${{ inputs.component }}"`. (9) 'Debug Infracost': `cat ${{ steps.vars.outputs.plan_file }}.json`. (10) 'Set Infracost Variables': `if [[ "${{ fromJson(...).enable-infracost }}" == "true" ]]`. (11) 'Terraform Apply': `atmos terraform apply ${{ inputs.component }} --stack ${{ inputs.stack }}`, `-var "job:${{ github.job }}"`, `--log-level $([[ "${{ inputs.debug }}" == "true" ]])`, `--output "${{ github.workspace }}/atmos-apply-summary.md"`.

Locations:

- `action.yml:65`
- `action.yml:151`
- `action.yml:158`
- `action.yml:166`
- `action.yml:174`
- `action.yml:250`
- `action.yml:258`
- `action.yml:265`
- `action.yml:276`
- `action.yml:284`
- `action.yml:300`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs or step outputs to $GITHUB_ENV and $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Violations: (1) 'Set atmos cli config path vars': `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — attacker-controlled inputs.atmos-config-path written to GITHUB_ENV unsanitized. (2) 'Define Job Control State Variables': `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV` — inputs.debug written to GITHUB_ENV unsanitized. (3) 'Set atmos cli base path vars': `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` then `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV` — step output written to GITHUB_ENV unsanitized. (4) 'Define Job Variables': variables derived from ${{ inputs.stack }}, ${{ inputs.component }}, ${{ inputs.sha }}, and step outputs written to GITHUB_OUTPUT via `echo "stack_name=$STACK_NAME" >> $GITHUB_OUTPUT` etc., without sanitization.

Locations:

- `action.yml:65`
- `action.yml:151`
- `action.yml:167`
- `action.yml:181`

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

Fixed all security findings in action.yml:

1. unpinned-uses: Pinned all 13 uses: references to full 40-character SHA hashes with original tags preserved as comments. Used lookup_action_sha to resolve real SHAs for: actions/setup-node@v4, actions/checkout@v4, cloudposse/github-action-setup-atmos@v2, cloudposse/github-action-atmos-get-setting@v2, hashicorp/setup-terraform@v3, cloudposse-github-actions/install-gh-releases@v1, aws-actions/configure-aws-credentials@v4 (x3), cloudposse/github-action-terraform-plan-storage@v1 (x2), infracost/actions/setup@v3, actions/cache@v4.

2. script-injection and static-inline-injection: Moved all ${{ }} expressions out of run: blocks into env: maps for every affected step: Set atmos cli config path vars, Define Job Control State Variables, Check If GitHub Actions is Enabled, Set atmos cli base path vars, Define Job Variables, Check Whether Infracost is Enabled, Convert PLANFILE to JSON, Generate Infracost Diff, Debug Infracost, Set Infracost Variables, Terraform Apply. Shell scripts now reference plain environment variables.

3. github-env-injection: Added printf '%s' "$VAR" | tr -d '\n\r' sanitization before writing user-controlled values to $GITHUB_ENV in Set atmos cli config path vars, Define Job Control State Variables, and Set atmos cli base path vars. Values written to $GITHUB_OUTPUT in Define Job Variables are also sanitized via printf+tr before use.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Define Job Variables' step in action.yml to sanitize all four values derived from untrusted step outputs before writing them to $GITHUB_OUTPUT. Added `printf '%s' ... | tr -d '\n\r'` sanitization for: (1) COMPONENT_PATH_INPUT before passing to realpath, (2) COMPONENT_CACHE_KEY_INPUT before passing to basename, and (3) the resulting COMPONENT_PATH, COMPONENT_CACHE_KEY, PLAN_FILE, and LOCK_FILE values before writing to $GITHUB_OUTPUT. This prevents newline injection attacks that could poison $GITHUB_OUTPUT with arbitrary key=value pairs.

