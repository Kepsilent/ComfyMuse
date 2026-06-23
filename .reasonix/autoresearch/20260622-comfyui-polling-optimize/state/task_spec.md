{
  "task": "comfyui-polling-optimize",
  "created": "2026-06-22",
  "goal": "Optimize ComfyUI polling: replace hardcoded 'every 5s' with step-count-based estimated wait time",
  "scope": "SKILL.md polling logic +流程图 polling section",
  "non_goals": ["change ComfyUI behavior", "add external dependencies"],
  "success_criteria": [
    "SKILL.md polling text references step count for estimated wait",
    "Polling logic uses formula rather than fixed interval",
    "Flow chart同步更新"
  ],
  "verification_gates": [
    "grep SKILL.md for old '每5s' text - should be gone",
    "grep flowchart for same - should be updated"
  ]
}
