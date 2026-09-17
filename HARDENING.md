<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v7.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-apply/v7.1.0** was hardened automatically. 41 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ ... }} expressions inside shell commands (sub-rule a), allowing script injection. Affected steps include:

1. 'Set atmos cli config path vars': `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — inputs.atmos-config-path interpolated directly in shell.

2. 'Define Job Control State Variables': `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV` — inputs.debug interpolated directly.

3. 'Check If GitHub Actions is Enabled For Component': `if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).github-actions-enabled }}" == "true" ...` — steps output interpolated directly.

4. 'Set atmos cli base path vars': `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` — steps output interpolated directly.

5. 'Define Job Variables': `STACK_NAME=$(echo "${{ inputs.stack }}" | sed ...)`, `COMPONENT_PATH=$( realpath ${{ fromJson(...).component-path }})`, `COMPONENT_NAME=$(echo "${{ inputs.component }}" | sed ...)`, `RETRIEVED_PLAN_FILENAME="...-${{ inputs.sha }}.planfile"` — multiple inputs interpolated directly.

6. 'Plan prepare': `if [[ -n "${{ inputs.identity }}" ]]`, `base_cmd+=" --identity=${{ inputs.identity }}"`, `-var "target:${{ inputs.stack }}-${{ inputs.component }}"`, `-var "job:${{ github.job }}"`, `atmos terraform plan ${{ inputs.component }} --stack ${{ inputs.stack }}`, `-out=${{ steps.vars.outputs.renewed_plan_file }}` — multiple inputs and github context interpolated directly.

7. 'Terraform Apply': `atmos terraform deploy ${{ inputs.component }} --stack ${{ inputs.stack }}`, `--planfile ${{ steps.plan-file.outputs.plan_filename }}`, `atmos terraform output ${{ inputs.component }} --stack ${{ inputs.stack }}`, `1> ${{ steps.vars.outputs.component_path }}/output_values.json` — multiple inputs and step outputs interpolated directly.

8. 'Convert PLANFILE to JSON': `${{ fromJson(steps.atmos-settings.outputs.settings).command }} show -json "${{ steps.plan-file.outputs.plan_file }}"` — step outputs used as command and argument.

All of these allow an attacker-controlled value to be interpreted as shell code.

Locations:

- `action.yml:68`
- `action.yml:75`
- `action.yml:218`
- `action.yml:226`
- `action.yml:237`
- `action.yml:249`
- `action.yml:253`
- `action.yml:258`
- `action.yml:263`
- `action.yml:268`
- `action.yml:350`
- `action.yml:360`
- `action.yml:365`
- `action.yml:370`
- `action.yml:375`
- `action.yml:380`
- `action.yml:385`
- `action.yml:390`
- `action.yml:490`
- `action.yml:510`
- `action.yml:515`
- `action.yml:520`
- `action.yml:525`
- `action.yml:530`

### github-env-injection (severity: high)

Multiple run: blocks write untrusted input values directly to $GITHUB_ENV or $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'):

1. 'Set atmos cli config path vars' (line ~68): `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — inputs.atmos-config-path (caller-controlled) written to GITHUB_ENV without sanitization.

2. 'Define Job Control State Variables' (line ~218): `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV` — inputs.debug written to GITHUB_ENV without sanitization.

3. 'Set atmos cli base path vars' (line ~237): `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` then `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV` — step output (derived from caller-controlled component settings) written to GITHUB_ENV without sanitization.

4. 'Define Job Variables' (line ~249): Multiple values derived from ${{ inputs.stack }}, ${{ inputs.component }}, ${{ inputs.sha }}, and step outputs are written to $GITHUB_OUTPUT (e.g., `echo "stack_name=$STACK_NAME" >> $GITHUB_OUTPUT`, `echo "retrieved_plan_filename=$RETRIEVED_PLAN_FILENAME" >> $GITHUB_OUTPUT`) without sanitization.

5. 'Determine Plan File' (line ~430): `echo "plan_file=${{ steps.vars.outputs.retrieved_plan_file }}" >> $GITHUB_OUTPUT` and similar — step outputs written to GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:68`
- `action.yml:218`
- `action.yml:237`
- `action.yml:249`
- `action.yml:430`

### unpinned-uses (severity: high)

All 13 uses: references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved:

- uses: actions/setup-node@v4
- uses: actions/checkout@v7
- uses: cloudposse/github-action-setup-atmos@v2
- uses: cloudposse/github-action-atmos-get-setting@v2
- uses: hashicorp/setup-terraform@v3
- uses: cloudposse-github-actions/install-gh-releases@v1
- uses: aws-actions/configure-aws-credentials@v4 (used 3 times)
- uses: cloudposse/github-action-terraform-plan-storage@v1 (used 2 times)
- uses: actions/cache@v4
- uses: infracost/actions/setup@v3

All should be pinned to full 40-character commit SHAs.

Locations:

- `action.yml:62`
- `action.yml:66`
- `action.yml:72`
- `action.yml:78`
- `action.yml:196`
- `action.yml:202`
- `action.yml:213`
- `action.yml:302`
- `action.yml:316`
- `action.yml:340`
- `action.yml:380`
- `action.yml:408`
- `action.yml:460`

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

Rewrote hardened/action/action.yml with all security fixes:

1. **unpinned-uses**: Pinned all 13 action references to full 40-char SHA digests (actions/setup-node, actions/checkout, cloudposse/github-action-setup-atmos, cloudposse/github-action-atmos-get-setting, hashicorp/setup-terraform, cloudposse-github-actions/install-gh-releases, aws-actions/configure-aws-credentials x3, cloudposse/github-action-terraform-plan-storage x2, actions/cache, infracost/actions/setup).

2. **script-injection / static-inline-injection**: Moved all ${{ }} expressions out of run: blocks into env: blocks for every affected step (Set atmos cli config path vars, Define Job Control State Variables, Check If GitHub Actions is Enabled, Set atmos cli base path vars, Define Job Variables, Plan prepare, Determine Plan File, Convert PLANFILE to JSON, Generate Infracost Diff, Debug Infracost, Set Infracost Variables, Terraform Apply). Shell scripts now reference plain environment variables.

3. **github-env-injection**: All values derived from inputs or step outputs that are written to $GITHUB_ENV or $GITHUB_OUTPUT are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before writing, preventing newline injection attacks.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in action.yml:
1. 'Plan prepare' step (line ~385): Converted `base_cmd` from a string to a bash array. Changed `base_cmd+=" --identity=$INPUT_IDENTITY"` to `base_cmd+=("--identity=$INPUT_IDENTITY")` and replaced all `${base_cmd}` unquoted expansions with `"${base_cmd[@]}"`.
2. 'Terraform Apply' step (line ~545): Same fix applied — `base_cmd` converted to a bash array with properly quoted element addition and `"${base_cmd[@]}"` expansion in all three command invocations (tfcmt, atmos terraform deploy, atmos terraform output). Also converted `SKIP_INIT_FLAG` from a string to an array for consistency.
These changes prevent an attacker controlling `inputs.identity` from injecting shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) to execute arbitrary commands.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Plan prepare' step (action.yml ~line 490). The original code used `sed -i "s/{{COMPONENT_NAME}}/${COMPONENT_NAME}/g"` and `sed -i "s/{{STACK_NAME}}/$INPUT_STACK/g"` where caller-controlled values were expanded inside double-quoted shell strings. A value containing `"` would terminate the outer shell string enabling arbitrary command injection; a value containing `/` would break the sed delimiter. Fixed by replacing both sed substitutions with a single `perl -i -pe` command that passes the untrusted values as environment variables (`COMPONENT_NAME="$INPUT_COMPONENT" STACK_NAME="$INPUT_STACK" perl -i -pe 's/\Q{{COMPONENT_NAME}}\E/$ENV{COMPONENT_NAME}/g; s/\Q{{STACK_NAME}}\E/$ENV{STACK_NAME}/g'`). Perl's `\Q...\E` quotes the pattern literally and `$ENV{VAR}` reads replacement values from the environment, so no shell interpretation of the values occurs regardless of their content.

