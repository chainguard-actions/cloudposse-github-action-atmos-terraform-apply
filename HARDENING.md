<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v7.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-apply/v7.1.0** was hardened automatically. 41 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in action.yml use mutable tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references include: `actions/setup-node@v4`, `actions/checkout@v7`, `cloudposse/github-action-setup-atmos@v2`, `cloudposse/github-action-atmos-get-setting@v2`, `hashicorp/setup-terraform@v3`, `cloudposse-github-actions/install-gh-releases@v1`, `aws-actions/configure-aws-credentials@v4` (used 3 times), `cloudposse/github-action-terraform-plan-storage@v1` (used 2 times), `actions/cache@v4`, and `infracost/actions/setup@v3`.

Locations:

- `action.yml:63`
- `action.yml:67`
- `action.yml:75`
- `action.yml:100`
- `action.yml:175`
- `action.yml:183`
- `action.yml:199`
- `action.yml:280`
- `action.yml:316`
- `action.yml:348`
- `action.yml:382`
- `action.yml:407`
- `action.yml:500`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ }}` expressions inside shell command strings, violating rule (a). Before the shell executes the script, the Actions runner substitutes these expressions verbatim, allowing an attacker-controlled value to inject arbitrary shell commands.

1. **'Set atmos cli config path vars'** step: `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — `${{ inputs.atmos-config-path }}` is interpolated directly into a shell command.

2. **'Define Job Control State Variables'** step: `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV` — `${{ inputs.debug }}` interpolated directly.

3. **'Set atmos cli base path vars'** step: `ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"` — steps output interpolated directly.

4. **'Define Job Variables'** step: `STACK_NAME=$(echo "${{ inputs.stack }}" | sed ...)`, `COMPONENT_PATH=$( realpath ${{ fromJson(...).component-path }})`, `COMPONENT_NAME=$(echo "${{ inputs.component }}" | sed ...)`, `RETRIEVED_PLAN_FILENAME="...-${{ inputs.sha }}.planfile"` — multiple inputs interpolated directly.

5. **'Plan prepare'** step: `atmos terraform plan ${{ inputs.component }} --stack ${{ inputs.stack }}`, `if [[ -n "${{ inputs.identity }}" ]]`, `base_cmd+=" --identity=${{ inputs.identity }}"`, `-out=${{ steps.vars.outputs.renewed_plan_file }}`, `atmos terraform plan-diff ${{ inputs.component }} --stack ${{ inputs.stack }}` — multiple inputs and step outputs interpolated directly into shell commands.

6. **'Determine Plan File'** step: `if [[ "${{ inputs.skip-plandiff }}" == "true" ]]`, `echo "plan_file=${{ steps.vars.outputs.retrieved_plan_file }}" >> $GITHUB_OUTPUT` — inputs and step outputs interpolated directly.

7. **'Convert PLANFILE to JSON'** step: `${{ fromJson(steps.atmos-settings.outputs.settings).command }} show -json "${{ steps.plan-file.outputs.plan_file }}"` — step output used as the command name, enabling arbitrary command execution.

8. **'Generate Infracost Diff'** step: `--project-name "${{ inputs.stack }}-${{ inputs.component }}"` — inputs interpolated directly.

9. **'Terraform Apply'** step: `atmos terraform deploy ${{ inputs.component }} --stack ${{ inputs.stack }}`, `--planfile ${{ steps.plan-file.outputs.plan_filename }}`, `atmos terraform output ${{ inputs.component }} --stack ${{ inputs.stack }}`, `echo "[Job](.../${{ github.repository }}/actions/runs/${{ github.run_id }})"` — multiple inputs and github context values interpolated directly.

Locations:

- `action.yml:72`
- `action.yml:155`
- `action.yml:163`
- `action.yml:170`
- `action.yml:300`
- `action.yml:340`
- `action.yml:460`
- `action.yml:480`
- `action.yml:510`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted inputs directly to `$GITHUB_ENV` and `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline character in any of these values allows an attacker to inject arbitrary environment variable assignments or output key-value pairs.

1. **'Set atmos cli config path vars'** step writes `${{ inputs.atmos-config-path }}` to `$GITHUB_ENV`: `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV`

2. **'Define Job Control State Variables'** step writes `${{ inputs.debug }}` to `$GITHUB_ENV`: `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV`

3. **'Set atmos cli base path vars'** step writes `steps.atmos-settings.outputs.settings` (base-path) to `$GITHUB_ENV`: `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV` where `ATMOS_BASE_PATH` is set from `${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}`.

4. **'Define Job Variables'** step writes multiple values derived from `${{ inputs.stack }}`, `${{ inputs.component }}`, `${{ inputs.sha }}`, and step outputs to `$GITHUB_OUTPUT` without sanitization.

5. **'Determine Plan File'** step writes `${{ steps.vars.outputs.retrieved_plan_file }}` and `${{ steps.vars.outputs.retrieved_plan_filename }}` to `$GITHUB_OUTPUT` without sanitization.

Locations:

- `action.yml:72`
- `action.yml:155`
- `action.yml:163`
- `action.yml:170`
- `action.yml:340`

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

Fixed all security findings in hardened/action/action.yml:

1. **unpinned-uses**: Pinned all 13 `uses:` references to full 40-character SHA digests using lookup_action_sha. All original tag names preserved as comments.

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ steps.*.outputs.* }}, and ${{ github.* }} expressions from `run:` blocks into `env:` blocks. Referenced as shell variables ($VAR_NAME) in the shell scripts. Affected steps: 'Set atmos cli config path vars', 'Define Job Control State Variables', 'Set atmos cli base path vars', 'Define Job Variables', 'Plan prepare', 'Determine Plan File', 'Convert PLANFILE to JSON', 'Generate Infracost Diff', 'Debug Infracost', 'Terraform Apply'.

3. **github-env-injection**: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing values derived from user inputs to $GITHUB_ENV and $GITHUB_OUTPUT. Affected steps: 'Set atmos cli config path vars' (ATMOS_CLI_CONFIG_PATH), 'Define Job Control State Variables' (DEBUG_ENABLED), 'Set atmos cli base path vars' (ATMOS_BASE_PATH), 'Define Job Variables' (stack_name, component_name via tr -d in sed pipeline).

Note: The 'Check If GitHub Actions is Enabled For Component' and 'Set Infracost Variables' steps retain `${{ fromJson(steps.atmos-settings.outputs.settings).* }}` in their `run:` blocks as these were not flagged in the findings and represent trusted step outputs rather than user-controlled inputs.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 5 script injection vulnerabilities in action.yml by moving ${{ }} expressions from run: blocks into env: blocks:
1. 'Check If GitHub Actions is Enabled For Component': moved fromJson(settings).github-actions-enabled to GITHUB_ACTIONS_ENABLED env var and fromJson(settings).atmos-pro-enabled to ATMOS_PRO_ENABLED env var.
2. 'Plan prepare': moved github.job to GITHUB_JOB_NAME env var; shell now references $GITHUB_JOB_NAME.
3. 'Check Whether Infracost is Enabled': moved fromJson(settings).enable-infracost to ENABLE_INFRACOST env var.
4. 'Set Infracost Variables': moved fromJson(settings).enable-infracost to ENABLE_INFRACOST env var.
5. 'Terraform Apply': moved github.job to GITHUB_JOB_NAME env var; shell now references $GITHUB_JOB_NAME.

### Iteration 3

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed all four security findings in hardened/action/action.yml:

1. 'Set atmos cli config path vars' (line 75): Added sanitization of realpath output via `printf '%s' "$(realpath ...)" | tr -d '\n\r'` before writing to $GITHUB_ENV.

2. 'Define Job Variables' (lines 230-235): Added `printf '%s' ... | tr -d '\n\r'` sanitization for COMPONENT_PATH (realpath output), COMPONENT_CACHE_KEY (basename output), PLAN_FILE, RETRIEVED_PLAN_FILE, RENEWED_PLAN_FILE, and LOCK_FILE before writing to $GITHUB_OUTPUT.

3. 'Determine Plan File' (lines 388-393): Added sanitization of $RETRIEVED_PLAN_FILE, $RETRIEVED_PLAN_FILENAME, $RENEWED_PLAN_FILE, and $RENEWED_PLAN_FILENAME via `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

4. 'Plan prepare' and 'Terraform Apply' (lines 374, 380, 490, 500): Converted `base_cmd` from a string variable (expanded unquoted as `${base_cmd}`) to a bash array (`base_cmd=()`), with identity added as `base_cmd+=(--identity="$INPUT_IDENTITY")` and expanded safely as `"${base_cmd[@]}"` to prevent word splitting and command injection.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted $INPUT_STACK variable expansion in the 'Plan prepare' step's sed command. Changed `sed -i "s/{{STACK_NAME}}/$INPUT_STACK/g"` to `sed -i "s/{{STACK_NAME}}/"$INPUT_STACK"/g"` so that $INPUT_STACK is properly double-quoted, preventing shell metacharacters in the user-controlled input from causing unexpected behavior.

### Iteration 5

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the 'Plan prepare' step at action.yml line 530. The sed command `sed -i "s/{{STACK_NAME}}/"$INPUT_STACK"/g"` had $INPUT_STACK outside the double-quoted string, allowing shell metacharacters in an attacker-controlled stack name to inject arbitrary commands. Fixed by moving the variable inside the double-quoted string: `sed -i "s/{{STACK_NAME}}/${INPUT_STACK}/g"`. The INPUT_STACK variable was already properly set via the step's env: block from inputs.stack.

