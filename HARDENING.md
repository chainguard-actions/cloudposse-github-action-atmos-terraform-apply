<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v7.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-apply/v7.1.0** was hardened automatically. 41 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 'uses:' references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references include: actions/setup-node@v4, actions/checkout@v7, cloudposse/github-action-setup-atmos@v2, cloudposse/github-action-atmos-get-setting@v2, hashicorp/setup-terraform@v3, cloudposse-github-actions/install-gh-releases@v1, aws-actions/configure-aws-credentials@v4 (used 3 times), cloudposse/github-action-terraform-plan-storage@v1 (used 2 times), actions/cache@v4, infracost/actions/setup@v3.

Locations:

- `action.yml:68`
- `action.yml:72`
- `action.yml:79`
- `action.yml:103`
- `action.yml:155`
- `action.yml:162`
- `action.yml:175`
- `action.yml:222`
- `action.yml:248`
- `action.yml:270`
- `action.yml:295`
- `action.yml:313`
- `action.yml:330`

### script-injection (severity: high)

Rule (a): Multiple run: blocks directly interpolate ${{ ... }} expressions inside shell command strings, allowing an attacker-controlled value to break out of the intended command context. Specific violations:

1. 'Set atmos cli config path vars' step: `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV` — inputs.atmos-config-path is interpolated directly into a shell command.

2. 'Define Job Variables' step: `STACK_NAME=$(echo "${{ inputs.stack }}" | sed ...)`, `COMPONENT_PATH=$( realpath ${{ fromJson(...).component-path }})`, `COMPONENT_NAME=$(echo "${{ inputs.component }}" | sed ...)`, `RETRIEVED_PLAN_FILENAME="$COMPONENT_SLUG-${{ inputs.sha }}.planfile"` — multiple inputs interpolated directly.

3. 'Plan prepare' step: `atmos terraform plan ${{ inputs.component }} --stack ${{ inputs.stack }}`, `base_cmd+=" --identity=${{ inputs.identity }}"`, `-var "target:${{ inputs.stack }}-${{ inputs.component }}"`, `-var "job:${{ github.job }}"`, `-out=${{ steps.vars.outputs.renewed_plan_file }}`, `atmos terraform plan-diff ${{ inputs.component }} --stack ${{ inputs.stack }}` — inputs and github context interpolated directly into shell.

4. 'Convert PLANFILE to JSON' step: `${{ fromJson(steps.atmos-settings.outputs.settings).command }} show -json "${{ steps.plan-file.outputs.plan_file }}"` — step output used as the command name itself.

5. 'Terraform Apply' step: `atmos terraform deploy ${{ inputs.component }} --stack ${{ inputs.stack }}`, `-var "target:${{ inputs.stack }}-${{ inputs.component }}"`, `-var "job:${{ github.job }}"`, `--planfile ${{ steps.plan-file.outputs.plan_filename }}`, `atmos terraform output ${{ inputs.component }} --stack ${{ inputs.stack }}`, `cd ${{ steps.vars.outputs.component_path }}` — multiple inputs and step outputs interpolated directly.

Locations:

- `action.yml:76`
- `action.yml:196`
- `action.yml:197`
- `action.yml:198`
- `action.yml:199`
- `action.yml:200`
- `action.yml:201`
- `action.yml:202`
- `action.yml:330`
- `action.yml:340`
- `action.yml:341`
- `action.yml:342`
- `action.yml:343`
- `action.yml:344`
- `action.yml:345`
- `action.yml:346`
- `action.yml:347`
- `action.yml:348`
- `action.yml:349`
- `action.yml:350`
- `action.yml:351`
- `action.yml:352`
- `action.yml:353`
- `action.yml:354`
- `action.yml:355`
- `action.yml:356`
- `action.yml:357`
- `action.yml:358`
- `action.yml:359`
- `action.yml:360`
- `action.yml:361`
- `action.yml:362`
- `action.yml:363`
- `action.yml:364`
- `action.yml:365`
- `action.yml:366`
- `action.yml:367`
- `action.yml:368`
- `action.yml:369`
- `action.yml:370`
- `action.yml:371`
- `action.yml:372`
- `action.yml:373`
- `action.yml:374`
- `action.yml:375`
- `action.yml:376`
- `action.yml:377`
- `action.yml:378`
- `action.yml:379`
- `action.yml:380`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs directly to $GITHUB_ENV or $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'):

1. 'Set atmos cli config path vars' step writes `${{ inputs.atmos-config-path }}` (a required user-supplied input) directly to $GITHUB_ENV: `echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV`. A newline in the input value can inject arbitrary environment variables.

2. 'Define Job Control State Variables' step writes `${{ inputs.debug }}` directly to $GITHUB_ENV: `echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV`.

3. 'Define Job Variables' step writes values derived from `${{ inputs.stack }}`, `${{ inputs.component }}`, `${{ inputs.sha }}`, and `${{ fromJson(steps.atmos-settings.outputs.settings).component-path }}` to $GITHUB_OUTPUT without sanitization (e.g. `echo "stack_name=$STACK_NAME" >> $GITHUB_OUTPUT`, `echo "retrieved_plan_filename=$RETRIEVED_PLAN_FILENAME" >> $GITHUB_OUTPUT`, etc.).

4. 'Set atmos cli base path vars' step writes `${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}` (a step output derived from user-controlled component/stack inputs) to $GITHUB_ENV: `echo "ATMOS_BASE_PATH=$(realpath ${ATMOS_BASE_PATH:-./})" >> $GITHUB_ENV`.

Locations:

- `action.yml:76`
- `action.yml:168`
- `action.yml:196`
- `action.yml:197`
- `action.yml:198`
- `action.yml:199`
- `action.yml:200`
- `action.yml:201`
- `action.yml:202`
- `action.yml:203`
- `action.yml:204`
- `action.yml:205`
- `action.yml:206`
- `action.yml:207`
- `action.yml:208`
- `action.yml:185`

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

All 13 uses: references pinned to full SHA digests. All ${{ inputs.* }} and ${{ github.job/run_id/repository }} expressions moved from run: blocks to env: blocks. Values written to $GITHUB_ENV/$GITHUB_OUTPUT sanitized with printf '%s' | tr -d '\n\r' to prevent newline injection. The Convert PLANFILE to JSON step now uses $TF_COMMAND env var instead of inline ${{ fromJson(...).command }} expression. The Terraform Apply step uses env vars for all inputs including component_path, plan_filename, and github context values.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 5 locations across 2 finding types:

1. 'Check If GitHub Actions is Enabled For Component' (line 235): Moved both `fromJson(steps.atmos-settings.outputs.settings).github-actions-enabled` and `.atmos-pro-enabled` expressions to env: block as GITHUB_ACTIONS_ENABLED and ATMOS_PRO_ENABLED. Added sanitization with `printf '%s' | tr -d '\n\r'` before using in shell condition. Fixes both script-injection and github-env-injection.

2. 'Check Whether Infracost is Enabled' (line 490): Moved `fromJson(steps.atmos-settings.outputs.settings).enable-infracost` to env: block as ENABLE_INFRACOST. Added sanitization before using in shell condition. Fixes both script-injection and github-env-injection.

3. 'Set Infracost Variables' (line 535): Moved `fromJson(steps.atmos-settings.outputs.settings).enable-infracost` to env: block as ENABLE_INFRACOST. Added sanitization before using in shell condition. Fixes script-injection.

4. 'Plan prepare' (line 430): Converted `base_cmd` from a string variable to a bash array (`base_cmd=()`), appending with `base_cmd+=("--identity=$INPUT_IDENTITY")`, and expanded with `"${base_cmd[@]}"` in all three usages. Fixes unquoted shell variable expansion.

5. 'Terraform Apply' (line 570): Added missing `INPUT_IDENTITY: ${{ inputs.identity }}` to env: block, built `base_cmd` as a bash array, converted `SKIP_INIT_FLAG` string to `skip_init_flag` array, and used `"${base_cmd[@]}"` and `"${skip_init_flag[@]}"` for all expansions. Fixes unquoted shell variable expansion and the undefined `base_cmd` bug.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed two locations in action.yml:

1. 'Define Job Variables' step: Added `printf '%s' ... | tr -d '\n\r'` sanitization to COMPONENT_PATH (from `realpath "$INPUT_COMPONENT_PATH"`), COMPONENT_CACHE_KEY (from `basename "$INPUT_COMPONENT_PATH"`), and all derived path variables (PLAN_FILE, RETRIEVED_PLAN_FILENAME, RETRIEVED_PLAN_FILE, RENEWED_PLAN_FILENAME, RENEWED_PLAN_FILE, LOCK_FILE) before they are written to $GITHUB_OUTPUT.

2. 'Determine Plan File' step: Added sanitization of $RETRIEVED_PLAN_FILE, $RETRIEVED_PLAN_FILENAME, $RENEWED_PLAN_FILE, and $RENEWED_PLAN_FILENAME into SAFE_* variables using `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Set atmos cli config path vars' step in action.yml (line 83). The original code called `realpath "$safe_path"` and embedded its output directly into the `echo` command writing to `$GITHUB_ENV` without sanitizing the realpath result. The fix now: (1) sanitizes the input before passing to realpath, and (2) wraps the realpath output in `printf '%s' ... | tr -d '\n\r'` before storing in `safe_path`, then writes `$safe_path` to `$GITHUB_ENV`. This matches the correct pattern already used for `ATMOS_BASE_PATH` elsewhere in the file.

