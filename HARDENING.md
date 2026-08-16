<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-atmos-terraform-apply/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-atmos-terraform-apply/v7.0.0** was hardened automatically. 45 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses multiple action references pinned to mutable version tags instead of immutable 40-character SHA digests. Failing references: actions/setup-node@v4, actions/checkout@v4, cloudposse/github-action-setup-atmos@v2, cloudposse/github-action-atmos-get-setting@v2, hashicorp/setup-terraform@v3, cloudposse-github-actions/install-gh-releases@v1, aws-actions/configure-aws-credentials@v4 (multiple), cloudposse/github-action-terraform-plan-storage@v1 (multiple), actions/cache@v4, infracost/actions/setup@v3.

Locations:

- `action.yml:63`
- `action.yml:67`
- `action.yml:75`
- `action.yml:82`
- `action.yml:196`
- `action.yml:204`
- `action.yml:213`
- `action.yml:302`
- `action.yml:330`
- `action.yml:348`
- `action.yml:380`
- `action.yml:545`

### unpinned-uses (severity: high)

Workflow files use action/workflow references pinned to mutable tags or branch names instead of immutable SHA digests. Failing references: branch.yml uses cloudposse/.github/.github/workflows/shared-github-action.yml@main; release.yml uses cloudposse/.github/.github/workflows/shared-release-branches.yml@main; scheduled.yml uses cloudposse/github-actions-workflows-terraform-module/.github/workflows/scheduled.yml@main; test-atmos_pro.yml uses actions/checkout@v4, cloudposse/github-action-atmos-terraform-plan@v5, nick-fields/assert-action@v2; test-basic.yml uses actions/checkout@v4, cloudposse/github-action-atmos-terraform-plan@v5; test-plan_diff.yml uses actions/checkout@v4, cloudposse/github-action-atmos-terraform-plan@v5; test-plan_fail.yml uses actions/checkout@v4; test-plan_storage_disabled.yml uses actions/checkout@v4.

Locations:

- `.github/workflows/branch.yml:12`
- `.github/workflows/release.yml:9`
- `.github/workflows/scheduled.yml:10`
- `.github/workflows/test-atmos_pro.yml:27`
- `.github/workflows/test-atmos_pro.yml:55`
- `.github/workflows/test-atmos_pro.yml:68`
- `.github/workflows/test-atmos_pro.yml:89`
- `.github/workflows/test-basic.yml:27`
- `.github/workflows/test-basic.yml:55`
- `.github/workflows/test-plan_diff.yml:28`
- `.github/workflows/test-plan_diff.yml:55`
- `.github/workflows/test-plan_fail.yml:28`
- `.github/workflows/test-plan_storage_disabled.yml:28`

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ ... }} expressions inside shell command strings (sub-rule a). This allows an attacker who controls the inputs to inject arbitrary shell commands. Affected steps and offending lines:
- 'Set atmos cli config path vars': echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV
- 'Define Job Control State Variables': echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV
- 'Check If GitHub Actions is Enabled For Component': if [[ "${{ fromJson(steps.atmos-settings.outputs.settings).github-actions-enabled }}" == "true" ...
- 'Set atmos cli base path vars': ATMOS_BASE_PATH="${{ fromJson(steps.atmos-settings.outputs.settings).base-path }}"
- 'Define Job Variables': STACK_NAME=$(echo "${{ inputs.stack }}" ...), COMPONENT_PATH=$( realpath ${{ fromJson(...).component-path }}), COMPONENT_NAME=$(echo "${{ inputs.component }}" ...), RETRIEVED_PLAN_FILENAME="...-${{ inputs.sha }}.planfile"
- 'Plan prepare': if [[ -n "${{ inputs.identity }}" ]], -var "target:${{ inputs.stack }}-${{ inputs.component }}", atmos terraform plan ${{ inputs.component }} --stack ${{ inputs.stack }}, -out=${{ steps.vars.outputs.renewed_plan_file }}
- 'Terraform Apply': atmos terraform deploy ${{ inputs.component }} --stack ${{ inputs.stack }}, --planfile ${{ steps.plan-file.outputs.plan_filename }}, echo "[Job](.../${{ github.repository }}/actions/runs/${{ github.run_id }})"

Locations:

- `action.yml:72`
- `action.yml:231`
- `action.yml:239`
- `action.yml:252`
- `action.yml:261`
- `action.yml:265`
- `action.yml:267`
- `action.yml:441`
- `action.yml:453`
- `action.yml:455`
- `action.yml:460`
- `action.yml:601`
- `action.yml:613`
- `action.yml:617`
- `action.yml:648`

### script-injection (severity: high)

run: blocks in test workflow files directly interpolate ${{ ... }} expressions inside shell command strings (sub-rule a). Offending patterns include: mkdir -p ${{ runner.temp }}, cp ./tests/${{ matrix.platform }}/atmos.yaml ${{ runner.temp }}/atmos.yaml, sed -i -e 's#__STORAGE_REGION__#${{ env.AWS_REGION }}#g', and echo "seed=${GITHUB_RUN_ID}-${GITHUB_RUN_ATTEMPT}-${{ matrix.platform }}" >> $GITHUB_OUTPUT. These allow workflow-controllable values (runner.temp, matrix.platform, env.AWS_REGION) to be interpolated directly into shell before the shell parses them.

Locations:

- `.github/workflows/test-basic.yml:30`
- `.github/workflows/test-basic.yml:31`
- `.github/workflows/test-basic.yml:34`
- `.github/workflows/test-atmos_pro.yml:33`
- `.github/workflows/test-atmos_pro.yml:34`
- `.github/workflows/test-atmos_pro.yml:37`
- `.github/workflows/test-plan_diff.yml:31`
- `.github/workflows/test-plan_diff.yml:32`
- `.github/workflows/test-plan_diff.yml:35`
- `.github/workflows/test-plan_fail.yml:31`
- `.github/workflows/test-plan_fail.yml:32`
- `.github/workflows/test-plan_storage_disabled.yml:31`
- `.github/workflows/test-plan_storage_disabled.yml:32`

### github-env-injection (severity: high)

The 'Set atmos cli config path vars' run: block writes the value of inputs.atmos-config-path directly to $GITHUB_ENV via: echo "ATMOS_CLI_CONFIG_PATH=$(realpath ${{ inputs.atmos-config-path }})" >> $GITHUB_ENV — without applying the required sanitization step (printf '%s' ... | tr -d '\n\r') before the write. A caller-controlled newline in the input value could inject arbitrary environment variables.

Locations:

- `action.yml:72`

### github-env-injection (severity: high)

The 'Define Job Control State Variables' run: block writes inputs.debug directly to $GITHUB_ENV via: echo "DEBUG_ENABLED=${{ inputs.debug }}" >> $GITHUB_ENV — without sanitization. A caller-controlled newline in the input value could inject arbitrary environment variables.

Locations:

- `action.yml:231`

### github-env-injection (severity: high)

In test workflow files, the run: block writes ${{ matrix.platform }} (a workflow-controllable value) directly to $GITHUB_OUTPUT via: echo "seed=${GITHUB_RUN_ID}-${GITHUB_RUN_ATTEMPT}-${{ matrix.platform }}" >> $GITHUB_OUTPUT — without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A newline in the matrix value could inject additional output variables.

Locations:

- `.github/workflows/test-basic.yml:43`
- `.github/workflows/test-atmos_pro.yml:46`
- `.github/workflows/test-plan_diff.yml:44`
- `.github/workflows/test-plan_fail.yml:43`
- `.github/workflows/test-plan_storage_disabled.yml:43`

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

Fixed all findings across action.yml and .github/workflows/ files:

1. **unpinned-uses**: Pinned all action references in action.yml to full 40-char SHA digests (setup-node, checkout, github-action-setup-atmos, github-action-atmos-get-setting, setup-terraform, install-gh-releases, configure-aws-credentials x3, github-action-terraform-plan-storage x2, cache, infracost/actions/setup). Pinned all workflow references in branch.yml, release.yml, scheduled.yml, and test-*.yml files.

2. **script-injection / static-inline-injection**: Moved all ${{ inputs.* }}, ${{ fromJson(...) }}, ${{ runner.temp }}, ${{ matrix.platform }}, and ${{ env.AWS_REGION }} expressions out of run: blocks and into env: maps. Shell scripts now reference plain environment variables.

3. **github-env-injection**: Added sanitization (printf '%s' ... | tr -d '\n\r') before writing user-controlled values to $GITHUB_ENV and $GITHUB_OUTPUT in action.yml (atmos-config-path, debug) and test workflow files (matrix.platform).

Note: ${{ github.job }} was left inline in run: blocks as it is a static workflow-defined value not controllable by external actors.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all four security findings in hardened/action/action.yml:

1. **github-env-injection** ('Set atmos cli base path vars' step, line ~228): Added `safe_base_path=$(printf '%s' "$(realpath ...)" | tr -d '\n\r')` before writing to $GITHUB_ENV.

2. **github-env-injection** ('Define Job Variables' step, line ~255): Added a `safe()` shell helper function and wrapped all 12 `echo "key=$VALUE" >> $GITHUB_OUTPUT` writes with `$(safe "$VAR")` to strip embedded newlines/carriage-returns.

3. **script-injection (a)** ('Plan prepare' step, line ~420): Moved `${{ github.job }}` out of the `run:` block into the `env:` block as `GITHUB_JOB_NAME: ${{ github.job }}`, then referenced `$GITHUB_JOB_NAME` in the shell script.

4. **script-injection (b)** ('Plan prepare' step): Changed `base_cmd` string variable to a `base_cmd_args` bash array and replaced all unquoted `${base_cmd}` expansions with properly quoted `"${base_cmd_args[@]}"` to prevent shell metacharacter injection from `inputs.identity`.

5. **script-injection (a)** ('Terraform Apply' step, line ~575): Same fix as #3 — moved `${{ github.job }}` to env block as `GITHUB_JOB_NAME`.

6. **script-injection (b)** ('Terraform Apply' step): Added `INPUT_IDENTITY: ${{ inputs.identity }}` to the env block (it was missing), added `base_cmd_args` array construction, and replaced all unquoted `${base_cmd}` expansions with `"${base_cmd_args[@]}"`.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in 5 workflow files (.github/workflows/test-basic.yml, test-atmos_pro.yml, test-plan_diff.yml, test-plan_fail.yml, test-plan_storage_disabled.yml). In each file, the `${{ secrets.TERRAFORM_STATE_BUCKET }}`, `${{ secrets.TERRAFORM_STATE_TABLE }}`, `${{ secrets.TERRAFORM_STATE_ROLE }}`, and `${{ secrets.TERRAFORM_APPLY_ROLE }}` expressions were moved from inline sed substitution strings into the step's `env:` block as named environment variables (TERRAFORM_STATE_BUCKET, TERRAFORM_STATE_TABLE, TERRAFORM_STATE_ROLE, TERRAFORM_APPLY_ROLE). The sed commands were updated from single-quoted strings (which prevented variable expansion) to double-quoted strings referencing the env vars (e.g., `sed -i -e "s#__STORAGE_BUCKET__#$TERRAFORM_STATE_BUCKET#g"`). This ensures GitHub Actions template substitution never injects secret values directly into shell command strings.

