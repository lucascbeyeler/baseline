# Opening a Pull Request

Every change to `lucascbeyeler.baseline` goes through a pull request (PR) to
`master`. Nothing is pushed to `master` directly: a merge there runs the
CircleCI pipeline and publishes the role to Ansible Galaxy.

## 1. Create a branch

```bash
git checkout master
git pull origin master
git checkout -b <short-description>   # e.g. fix-chrony-drift-file
```

Use one branch per change. Keep unrelated fixes in separate PRs.

## 2. Make the change

* Follow the conventions in [SPEC.md](SPEC.md): fully qualified module names
  (`ansible.builtin.*`, `community.general.*`), new variables prefixed with
  `baseline_`, per-OS values in `vars/main.yml`, user-facing defaults in
  `defaults/main.yml`.
* Keep the role idempotent and keep every SSH edit validated with
  `sshd -t -f %s`.
* Update the tests with the change:
  * new variable, template or per-family value → `tests/unit/main.yml`;
  * new behaviour on the host → `molecule/default/verify.yml` (and
    `molecule.yml` / `prepare.yml` if the scenario needs it).
* Update the documentation: `README.md` (features and variables) and
  `documents/SPEC.md` when the behaviour, inputs or guarantees change.

## 3. Run the checks locally

From the repository root, inside a virtual environment
(see [Development.md](../Development.md)):

```bash
pip install -r requirements.txt ansible-lint
ansible-galaxy collection install -r requirements.yml
printf '[defaults]\nroles_path=../\n' > ansible.cfg   # ignored by git

ansible-lint
ansible-playbook -i tests/inventory tests/test.yml --syntax-check
ansible-playbook tests/unit/main.yml

ansible-galaxy collection install community.docker
molecule test        # needs Docker
```

All of them must pass before you open the PR. These are the same checks CI
runs.

## 4. Commit

* Write the subject in the imperative, describing the effect, e.g.
  `Make the Molecule prepare step fail loudly instead of stalling`.
* Use the body to explain *why* when it is not obvious.
* Do not commit `ansible.cfg`, `venv/`, `.venv/` or `.ansible/` (they are in
  `.gitignore`).

```bash
git add <files>
git commit
git push -u origin <branch>
```

## 5. Open the PR

On GitHub, open a PR from your branch to `master`
(`https://github.com/lucascbeyeler/baseline/compare/master...<branch>`), or
with the GitHub CLI:

```bash
gh pr create --base master --head <branch> --title "<summary>" --body-file pr.md
```

Use this description layout:

```markdown
## Summary
What changes and why (link the issue, if any).

## Changes
- Bullet list of the files/behaviours changed.

## Testing
- [ ] ansible-lint
- [ ] Syntax check
- [ ] Unit tests (`tests/unit/main.yml`)
- [ ] Molecule (`molecule test`)

## Impact
Breaking changes, new variables or effects on lucascbeyeler.zimbra (or "None").
```

## 6. Review and merge

1. Wait for the CircleCI `lint` and `molecule` jobs to be green. If one
   fails, fix the cause and push to the same branch — do not open a new PR.
2. Address review comments with new commits on the branch, and answer each
   comment (fixed in `<commit>`, or why not).
3. If `master` moved and the PR conflicts, merge `master` into the branch and
   resolve the conflict; do not force-push a shared branch.
4. The maintainer merges the PR with a merge commit
   (`Merge pull request #N from ...`). The `ansible-galaxy` job then imports
   the new version.
5. Delete the branch after the merge.

## 7. Changes that affect the Zimbra role

`lucascbeyeler.zimbra` relies on this role for the hostname, `/etc/hosts`
and the server preparation. When a change alters those (variable names,
the hosts line, supported distributions), say so in the **Impact** section
and open the matching PR in
[lucascbeyeler/zimbra](https://github.com/lucascbeyeler/zimbra), linking
the two PRs to each other.
