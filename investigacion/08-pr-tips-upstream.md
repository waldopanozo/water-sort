# PR listo — limited level tips

## Estado preparado (sin commit)

- Branch: `feature/limited-level-tips`
- Upstream: `sidhant947/water-sort` (remote `upstream`)
- Fork: `waldopanozo/water-sort` (remote `origin`)
- `investigacion/` excluida vía `.git/info/exclude` (no entra al PR)

### Archivos del aporte

1. `lib/ui/features/game/view_models/game_view_model.dart`
2. `lib/ui/features/game/views/game_view.dart`
3. `lib/ui/features/home/views/settings_view.dart`

## Cuando quieras commitear tú

```bash
cd ~/work/opensource/water-sort
git status
git add \
  lib/ui/features/game/view_models/game_view_model.dart \
  lib/ui/features/game/views/game_view.dart \
  lib/ui/features/home/views/settings_view.dart

git commit -m "$(cat <<'EOF'
Limit in-game tips to 3 from level 10 and in random mode

Tips highlight the next good pour from the current board state.
Levels 1–9 have no tips; reset refills the counter. Settings
documents the behavior instead of the old Hint Helper toggle.
EOF
)"
```

## Después del commit (push + PR)

```bash
git push -u origin HEAD

gh pr create --repo sidhant947/water-sort --base main --head waldopanozo:feature/limited-level-tips \
  --title "Limit in-game tips to 3 from level 10 (and random)" \
  --body "$(cat <<'EOF'
## Summary
- Campaign levels **1–9**: no tip button
- Campaign **level ≥ 10** and **random mode**: **3 tips** per puzzle
- Each tip uses the existing solver to highlight the next good pour from the **current** board (not tutorial text)
- Failed tip (no solution found) does **not** consume a tip
- **Reset** refills tips to 3
- Settings: replaced the old Hint Helper toggle with a short explanation of the tip rules

## Test plan
- [ ] Levels 1–9: tip button hidden
- [ ] Level 10+: button shows `TIP 3` → decreases on success
- [ ] At `TIP 0`: button disabled
- [ ] Reset level: counter back to 3
- [ ] Random mode: same 3-tip behavior
- [ ] Tip with no solvable path: snackbar, counter unchanged
- [ ] Settings → Gameplay shows LEVEL TIPS info (no toggle)

EOF
)"
```

O dime cuando hayas hecho el commit y yo ejecuto push + `gh pr create`.
