---
layout: post
category: Software Engineering
---

# Tool Call Component

`self._openclaw_session_phase: str = "idle"`

## Five Property Functions

```
is_openclaw_running_background
returns self.is_openclaw_running_background()

is_openclaw_animation_suppressed
i) check _openclaw_session_phase is in {"candiate", "active"}
ii) if _openclaw_session_phase == "candiate" or "active" [true]
iii) if _openclaw_session_phase == "idle" [false]

begin_openclaw_candidate
i) if current == idle
ii) set to self._openclaw_session_phase = "candidate"

confirm_openclaw_active
i) if current == idle or active
ii) then promote to self._openclaw_session_phase = "active"

clear_openclaw_session
set _openclaw_session_phase = "idle"
```

## First Shield

```python
_get_tools_for_trigger

if self.is_openclaw_animation_suppressed():
	# do not allow animations in tool_defs
	tool_defs = [t for t in tool_defs if getattr(t, "category", None) != "animation"]
```

```
_run_final_bg

- final_user_turn means run on the final user transcript
- self.begin_openclaw_candidate() is called right before scheduling the background task
- animation suppression starts immediately if OpenClaw is one of the final_user_turn tools
- pre-emptive gate because actual selection of openclaw takes time later in the selector LLM during _run_tool_call()
```

## Second Shield

```
_execute_tool_calls

- execution-time safety gate
- Even if somehow an animation tool call is leaked, do not call at execution time
```

## Setting State Permanently

```python
_run_for_category

# first, clear the candidate state if the critic result is None
if critic_result is None:
    if getattr(self, "_openclaw_session_phase", "idle") == "candidate":
        self.clear_openclaw_session()
        
# second, check the selected tool
selected_openclaw = any(
	((tc.get("function") or {}).get("name") or "").strip().lower() 
		== "openclaw" 
	for tc in tool_calls 
)

if getattr(self, "_openclaw_session_phase", "idle") == "candidate":
	if selected_openclaw:
		# openclaw tool selected mark as active
		self.confirm_openclaw_active()
	else:
		# openclaw tool not selected, clear the state
		self.clear_openclaw_session()
```

# Openclaw Component

```python
_confirm_openclaw_active # forwarding wrapper
_clear_openclaw_session # forwarding wrapper
_finish_openclaw_session 
	# call animation service and set back to idle
	# clear openclaw session
```

```python
# start animation on execution start
# steering path is no-op
if animation_service:
	self._logger.info("[OpenClaw] Animation -> Working"
	await animation_service.trigger(
		AnimationBridgeMessage(characterState="Working"),
		source="openclaw",
	)

animation_started = True
```

```python
# always clean up
# but only if animaiton transition happend and animation service exists
finally:
	await self._finish_openclaw_session(
		animation_service=animation_service,
		animation_started=animation_started,
	)
```