---
name: debug-methodology
description: Systematic debugging methodology for diagnosing and fixing bugs in running services and projects. Activate when encountering unexpected errors, service failures, regression bugs, deployment issues, or when a fix attempt has failed twice. Prevents common anti-patterns like patch-chaining, wrong-environment restarts, and "drunk man" random fixes.
---

# Debug Methodology

Systematic approach to debugging, distilled from real production incidents and industry best practices (Nicole Tietz, Brendan Gregg, Julia Evans).

## The Golden Rule

**Understand the system before touching the code.** Skipping this step is the #1 cause of debugging spirals.

## Phase 1: STOP — Assess Before Acting

Before ANY fix attempt:

```
□ What is the EXACT symptom? (error message, behavior, screenshot)
□ When did it last work? What changed since then?
□ How is the service running? (process, env, startup command)
```

For running services specifically:
```bash
# Check how the process was started
ps -p <PID> -o command=
# Check for virtual environments
ls .venv/ venv/ env/
# Check which interpreter/runtime
which python3 && python3 --version
which node && node --version
# Check environment variables
cat .env 2>/dev/null
```

**NEVER restart a service without first recording its original startup command.**

## Phase 2: Hypothesize — Form ONE Theory

Ask: "What is the simplest explanation?"

Priority order:
1. **Did I change something?** → diff/revert my changes first
2. **Did the environment change?** → check versions, deps, configs
3. **Did external inputs change?** → check API responses, data formats
4. **Is it a genuine new bug?** → only consider this after ruling out 1-3

## Phase 3: Test — Verify the Hypothesis

One change at a time. Verify before proceeding.

```
Change X → Test → Works? → Done
                → Fails? → REVERT X, form new hypothesis
```

**Do NOT stack changes.** If you changed 3 things and it works, you don't know which one fixed it (and the other 2 might cause future bugs).

## Phase 4: Patch-Chain Detection

**If you've made 2 fix attempts and it's still broken → STOP.**

This is the "patch chain" danger signal. You are likely:
- Fixing symptoms of a wrong fix, not the real problem
- In the wrong environment/context entirely
- Misunderstanding the system architecture

Action: **Revert ALL changes. Go back to Phase 1.**

## Anti-Patterns to Avoid

### 🚨 Drunk Man Anti-Pattern
Randomly changing things until the problem disappears.
→ Each change must have a specific hypothesis behind it.

### 🚨 Streetlight Anti-Pattern
Looking where you're comfortable, not where the problem is.
→ "Is this where the bug IS, or where I KNOW HOW TO LOOK?"

### 🚨 Cargo Cult Fix
Copying a fix from a similar-looking problem without understanding why it works.
→ Understand the mechanism before applying.

### 🚨 Ignoring the User
User says "it broke after you changed X" → immediately diff X, don't keep guessing.
→ User observations are the most valuable debugging data.

## Environment Checklist

Before modifying any service:

```
□ Runtime: which python/node/java? System or venv/nvm?
□ Dependencies: pip freeze / npm ls — match expected versions?
□ Config: .env, config.json, nginx — any recent changes?
□ Process manager: PM2/systemd/supervisor — how does it restart?
□ Logs: where are they? tail -f before reproducing
□ Backup: snapshot current state before any change
```

## Deployment Safety

```
□ Pull latest from server (don't overwrite blindly)
□ Backup current working version
□ Make changes locally
□ Test locally if possible
□ Deploy with same startup method as original
□ Verify immediately after deploy
□ If broken → revert to backup, THEN debug
```

## Quick Reference Decision Tree

```
Error appears
  ├─ Was I just editing? → DIFF my changes → REVERT if suspect
  ├─ Service won't start? → CHECK startup command + environment
  ├─ New error after fix? → STOP (patch chain!) → Revert all → Phase 1
  ├─ User reports regression? → DIFF before/after their last known-good
  └─ Intermittent? → CHECK logs + external dependencies + timing
```
