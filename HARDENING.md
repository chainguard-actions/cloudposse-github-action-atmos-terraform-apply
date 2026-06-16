<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **cloudposse--github-action-atmos-terraform-apply/v7.0.0** was hardened automatically. 41 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ ... }} expressions into shell commands (sub-rule a), allowing script injection by any caller of the action. Key violations: (1) 'Set atmos cli config path vars': echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV — inputs.atmos-config-path interpolated directly. (2) 'Define Job Control State Variables': echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV. (3) 'Check If GitHub Actions is Enabled': if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).github-actions-enabled }}" == "true" ]]. (4) 'Set atmos cli base path vars': ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}". (5) 'Define Job Variables': STACK_NAME=$(echo "${{ inputs.stack }}" | sed ...), COMPONENT_PATH=$( realpath ${{ fromJson(...).component-path }}), RETRIEVED_PLAN_FILENAME="$COMPONENT_SLUG-${{ inputs.sha }}.planfile". (6) 'Plan prepare': if [[ -n "${{ inputs.identity }}" ]], base_cmd+=" --identity=${{ inputs.identity }}", -var "target:${{ inputs.stack }}-${{ inputs.component }}", -var "job:${{ github.job }}", atmos terraform plan ${{ inputs.component }} --stack ${{ inputs.stack }}. (7) 'Determine Plan File': if [[ "${{ inputs.skip-plandiff }}" == "true" ]]. (8) 'Check Whether Infracost is Enabled': if [[ "${{ fromJson(...).enable-infracost }}" == "true" ]]. (9) 'Convert PLANFILE to JSON': ${{ fromJson(...).command }} show -json "${{ steps.plan-file.outputs.plan_file }}" — steps output used as the shell command itself. (10) 'Generate Infracost Diff': --project-name "${{ inputs.stack }}-${{ inputs.component }}". (11) 'Debug Infracost': cat ${{ steps.vars.outputs.plan_file }}.json. (12) 'Set Infracost Variables': if [[ "${{ fromJson(...).enable-infracost }}" == "true" ]]. (13) 'Terraform Apply': ${{ inputs.skip-plandiff }}, ${{ inputs.stack }}, ${{ inputs.component }}, ${{ github.job }}, ${{ github.repository }}, ${{ github.run_id }} all interpolated directly into shell.

Locations:

- `action.yml:72`
- `action.yml:155`
- `action.yml:162`
- `action.yml:170`
- `action.yml:177`
- `action.yml:248`
- `action.yml:302`
- `action.yml:316`
- `action.yml:330`
- `action.yml:340`
- `action.yml:356`
- `action.yml:365`
- `action.yml:380`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from caller-controlled inputs or steps outputs to $GITHUB_ENV or $GITHUB_OUTPUT without the required sanitization (printf '%s' ... | tr -d '\n\r'): (1) 'Set atmos cli config path vars': echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV — inputs.atmos-config-path written to GITHUB_ENV unsanitized. (2) 'Define Job Control State Variables': echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV — inputs.debug written to GITHUB_ENV unsanitized. (3) 'Set atmos cli base path vars': echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV — ATMOS_BASE_PATH sourced from steps output written to GITHUB_ENV unsanitized. (4) 'Define Job Variables': echo "stack_name=$STACK_NAME" >> $GITHUB_OUTPUT and echo "retrieved_plan_filename=$RETRIEVED_PLAN_FILENAME" >> $GITHUB_OUTPUT — values derived from inputs.stack, inputs.component, inputs.sha written to GITHUB_OUTPUT unsanitized. (5) 'Determine Plan File': echo "plan_file=${{ steps.vars.outputs.retrieved_plan_file }}" >> $GITHUB_OUTPUT — steps output (derived from inputs) written to GITHUB_OUTPUT unsanitized. (6) 'Set Infracost Variables': echo "infracost_details_diff_breakdown=$INFRACOST_DETAILS_DIFF_BREAKDOWN" >> "$GITHUB_OUTPUT" — external tool output written to GITHUB_OUTPUT unsanitized.

Locations:

- `action.yml:72`
- `action.yml:155`
- `action.yml:170`
- `action.yml:177`
- `action.yml:302`
- `action.yml:365`

### unpinned-uses (severity: high)

All uses: references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks. Unpinned references: uses: actions/setup-node@v4, uses: actions/checkout@v4, uses: cloudposse/github-action-setup-atmos@v2, uses: cloudposse/github-action-atmos-get-setting@v2, uses: hashicorp/setup-terraform@v3, uses: cloudposse-github-actions/install-gh-releases@v1, uses: aws-actions/configure-aws-credentials@v4 (3 occurrences), uses: cloudposse/github-action-terraform-plan-storage@v1 (2 occurrences), uses: actions/cache@v4, uses: infracost/actions/setup@v3.

Locations:

- `action.yml:65`
- `action.yml:69`
- `action.yml:76`
- `action.yml:82`
- `action.yml:131`
- `action.yml:136`
- `action.yml:143`
- `action.yml:196`
- `action.yml:210`
- `action.yml:226`
- `action.yml:237`
- `action.yml:322`

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

Rewrote action.yml with all security fixes applied:

1. UNPINNED-USES: Pinned all 12 action references to full 40-char SHA digests (actions/setup-node, actions/checkout, cloudposse/github-action-setup-atmos, cloudposse/github-action-atmos-get-setting, hashicorp/setup-terraform, cloudposse-github-actions/install-gh-releases, aws-actions/configure-aws-credentials x3, cloudposse/github-action-terraform-plan-storage x2, actions/cache, infracost/actions/setup).

2. SCRIPT-INJECTION / STATIC-INLINE-INJECTION: Moved all ${{ inputs.* }}, ${{ github.* }}, ${{ steps.* }}, and ${{ fromJson(...) }} expressions out of run: blocks and into env: blocks. Shell scripts now reference plain environment variables ($VAR_NAME) instead of interpolated expressions.

3. GITHUB-ENV-INJECTION: All values derived from caller-controlled inputs or step outputs that are written to $GITHUB_ENV or $GITHUB_OUTPUT are now sanitized using `printf '%s' "$VAR" | tr -d '\n\r'` before writing, preventing newline injection attacks.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection vulnerabilities in action.yml:

1. **$INPUT_IDENTITY / base_cmd injection (line ~330)**: In both 'Plan prepare' and 'Terraform Apply' steps, converted `base_cmd` from an unquoted string variable to a bash array (`base_cmd=()`). The identity value is now appended as a properly quoted array element (`base_cmd+=("--identity=$INPUT_IDENTITY")`), and all expansions use `"${base_cmd[@]}"` to prevent word-splitting and shell metacharacter injection. Also added `INPUT_IDENTITY: ${{ inputs.identity }}` to the Terraform Apply step's `env:` block (it was missing entirely).

2. **sed metacharacter injection (line ~370)**: Added proper escaping of sed replacement metacharacters before using `$INPUT_STACK` and `$INPUT_COMPONENT` in sed substitution commands. `SAFE_COMPONENT_NAME` escapes `&` and `\` via `sed 's/[&\]/\&/g'`. `SAFE_STACK_NAME` escapes `&`, `\`, and `/` (the sed delimiter) via `sed 's/[&\]/\&/g; s|/|\\/|g'`. These safe variables are used in the sed commands instead of the raw input values.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the 'Determine Plan File' step in action.yml by moving all four ${{ steps.vars.outputs.* }} expressions (retrieved_plan_file, retrieved_plan_filename, renewed_plan_file, renewed_plan_filename) from the run: block into the step's env: block as VARS_RETRIEVED_PLAN_FILE, VARS_RETRIEVED_PLAN_FILENAME, VARS_RENEWED_PLAN_FILE, and VARS_RENEWED_PLAN_FILENAME. The shell script now references these as plain environment variables, eliminating the script-injection risk from direct ${{ }} interpolation into the shell command string.

