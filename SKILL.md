---
name: boss-chrome-desktop
description: Use when the user wants Codex to operate BOSS or BOSS Zhipin through desktop automation in the user's currently logged-in Chrome browser, explicitly reusing the real Chrome session instead of an isolated Playwright browser, opening BOSS maximized, processing unread contacts in bounded batches, extracting online resume summaries for recruiting review, or migrating this exact workflow to another computer by checking and installing the same required capabilities, including multi-monitor, small-screen, and DPI-sensitive desktop runs.
---

# BOSS Chrome Desktop

## Purpose

This skill records the desktop-first workflow for BOSS tasks that must reuse the user's real Chrome login state.

Do not use an isolated Playwright browser for this workflow. Use desktop automation against the user's current Chrome window.

## Boundaries

- The user must already be logged in, or must complete login, captcha, and any second-factor checks manually.
- Do not request, store, export, or share cookies, passwords, SMS codes, or session files.
- Do not bypass platform risk controls, captcha, account restrictions, or posting limits.
- Do not save raw online resume text. Output structured summaries for the current hiring task instead.
- Temporary screenshots or clipboard reads used for extraction must be cleared after the run when practical.
- Before any irreversible action such as publishing, sending, deleting, or purchasing, stop and ask the user to confirm the visible page state.

## Portability Preflight: Exact Capability Replication

Use this before running the workflow on a new computer, a new agent host, or a different automation tool.

The goal is 1:1 replication of the validated path. Check for the same capabilities that were used successfully; do not silently substitute an isolated browser, a different browser profile, a different OCR path, or a different resume-reading strategy.

Required capability manifest:

1. Real desktop Chrome is installed and can run as the user's normal browser.
2. The user can manually log in to BOSS in that real Chrome profile. The agent must not request or export cookies, passwords, SMS codes, or session files.
3. A desktop-control command layer equivalent to `uvx desktop-agent` is available, with app focus/list, window screenshot, region screenshot, mouse click, mouse scroll, keyboard hotkey, keyboard keyup, and screen-size commands.
4. The agent runtime can inspect local screenshots visually and read Chinese resume text from them. The validated long-resume path uses agent visual reading of screenshots; local OCR is optional unless explicitly added and tested.
5. The environment can create and delete temporary screenshot files and can clear the clipboard after transient copy attempts.
6. The agent can ask the user for installation and screen-control permission before installing or running desktop-control tools.

Capability check sequence:

1. Check Chrome visibility: list desktop windows and confirm the user's Chrome window can be focused.
2. Check screen control: get screen size, maximize Chrome, and take a window screenshot.
3. Check input control: run harmless keyboard and mouse checks only against a safe UI state.
4. Check screenshot reading: capture a small validation screenshot with Chinese text and confirm the agent can read it clearly.
5. Check cleanup: delete the validation screenshot and clear the clipboard.

If any required capability is missing, stop and ask the user for permission before installing anything. Prefer mirror-backed installs by default.

Permission prompt template:

```text
I need to install or enable the same desktop automation capability used by this BOSS workflow.

This may install uv and desktop-agent using a package mirror when possible, then request screen, keyboard, mouse, screenshot, temporary-file, and clipboard access.

It will operate only your currently logged-in Chrome window. It will not read or export cookies, passwords, SMS codes, or session files. Online resumes are processed as transient screenshots or clipboard reads, summarized, and then temporary files are cleared.

May I continue with the install and capability check?
```

Windows mirror-backed check/install pattern:

```powershell
uv --version
$env:UV_INDEX_URL = "https://pypi.tuna.tsinghua.edu.cn/simple"
uvx desktop-agent --help
uvx desktop-agent screen size
uvx desktop-agent app list
```

If `uv` is unavailable, ask the user to approve installing `uv` first with their preferred trusted installer or package manager, then rerun the mirror-backed `uvx desktop-agent` checks. Do not proceed with BOSS automation until the checks pass.

## Screen And Coordinate Portability

Use this when running on a different monitor, a smaller screen, a multi-monitor setup, or a different Windows scaling/DPI setting.

1. Treat screen size, desktop coordinate space, and current Chrome window bounds as separate facts. `screen size` can report the primary or virtual desktop size while the BOSS window screenshot may have a different region and scale.
2. After focusing and maximizing Chrome, take a window screenshot and use its returned region as the current coordinate basis. Do not reuse coordinates from another monitor or an earlier run.
3. Prefer visual or text location for named controls such as Communication, Unread, Online Resume, and close buttons. Use coordinates only after a current screenshot confirms the element position.
4. If coordinates are needed, choose them relative to the current window or resume panel, not the physical screen. Recompute after every maximize, monitor switch, browser zoom change, popup, or overlay close.
5. Avoid click points near candidate names, company names, magnifier icons, Continue Communication, send controls, request-resume buttons, contact-exchange buttons, and the right action rail.
6. When focusing the online resume before scrolling, prefer a blank margin inside the resume body panel. If no safe blank area is visible, avoid clicking and use the panel scrollbar or a verified keyboard scroll instead.
7. Verify every scroll by comparing the next screenshot with the previous one. If the visible content did not move, assume focus drifted and choose a different scroll method.
8. If a company information card or other non-business overlay opens accidentally, close it, revalidate the resume view, and continue only if no message, status, purchase, or contact action was triggered.
9. For drag operations, always move to the exact drag start first, then drag. Never drag from resume text unless the intent is text selection.

## Preflight: Normalize BOSS UI State

Run this before contact or resume processing when the current page state is uncertain.

1. Focus the real Chrome window and maximize it.
2. If BOSS is not open, run Flow 1.
3. If BOSS is open but not on the communication page, navigate directly to `https://www.zhipin.com/web/chat/index`.
4. Wait for the communication UI to load and confirm the left navigation, contact list, and conversation area are visible.
5. Close non-business popups that block the workflow, such as Windows client promotion, app download prompts, discount ads, onboarding tips, or other overlays. Do not click download, purchase, promotion, or consent-changing buttons.
6. If the page asks for login, captcha, risk verification, or account confirmation, stop and let the user handle it.

## Flow 1: Open BOSS In Current Chrome

Use this flow when the user asks to use desktop automation on the currently logged-in Chrome session, avoid isolated browsers, and open BOSS.

1. Use the desktop-control workflow to find or focus the user's existing Chrome window.
2. If Chrome is not open, launch normal Chrome, not a Playwright-managed browser.
3. Maximize the Chrome window before page inspection or navigation so BOSS toolbar and side-panel actions are visible.
4. Open a new tab or use the address bar in the focused Chrome window.
5. Navigate to the official BOSS web entry, normally `https://www.zhipin.com/`. If the entry point appears stale or redirects unexpectedly, verify the current official URL before continuing.
6. Wait for the page to load and visually confirm whether the user's logged-in state is present.
7. Report the visible state to the user without taking further account actions.

## Flow 2: Enter HR Recruiting Interface

Use this flow after BOSS is open in the user's real Chrome session.

1. Inspect the visible page state with a screenshot or reliable screen reading.
2. Branch A: if the top-right area shows a registration/login entry, click that entry once. Because the workflow relies on the user's existing Chrome cookies, do not type credentials or request passwords. Wait for the cookie-based login or redirect to complete, then inspect the page again.
3. Branch B: if the left navigation already shows the Job Management entry, treat the page as already being in the HR recruiting interface and take no further action.
4. If neither branch is visible, stop and report the unknown state instead of guessing.
5. Do not continue into posting, messaging, or other account actions unless the user gives the next explicit step.

## Flow 3: Open Candidate Communication

Use this flow only after Flow 2 confirms the page is in the HR recruiting interface.

1. Inspect the left navigation and locate the Communication entry.
2. Click the Communication entry once.
3. Wait for the page to enter the candidate communication interface, normally a URL like `/web/chat/index`.
4. If the Communication entry is unavailable or navigation does not settle, navigate directly to `https://www.zhipin.com/web/chat/index`.
5. Close non-business popups that block the contact list or resume panel.
6. Confirm that the Communication entry is selected and the chat/contact list area is visible.
7. Do not select a candidate, send a message, or open a conversation unless the user gives the next explicit step.

## Flow 4: Open First Unread Contact

Use this flow only after Flow 3 confirms the candidate communication interface is open.

1. Inspect the contact list area and locate the Unread filter above the list.
2. Click the Unread filter once.
3. Wait for the unread list to settle.
4. Click the first visible contact in the unread list.
5. Confirm that the conversation panel opens.
6. Do not send a message, click quick-reply prompts, request a resume, request a phone number, or mark additional contacts unless the user gives the next explicit step.
7. If no unread contacts are visible, stop and report that state.

## Flow 5: Open And Extract Online Resume

Use this flow only after one contact conversation is open and the user explicitly asks to inspect the online resume.

1. Locate the Online Resume button in the contact conversation interface.
2. Click the Online Resume button once and wait for the resume overlay or panel to load.
3. Confirm that the online resume content is visible before extracting anything.
4. Prefer transient text extraction from the visible resume panel. If copyable text is available, click inside the resume, use select-all/copy, and read the clipboard. If copy is unavailable, use screenshots and extract only the visible content.
5. Extract into a structured hiring summary by default: identity header, availability, desired role, work history, project history, education, certificates, skills/tags, key fit notes, and optional JD match notes when the user provides a job description or matching criteria.
6. Do not write raw resume text to files, upload it elsewhere, or share it outside the current hiring task unless the user explicitly asks and the action is permitted.
7. Do not click Continue Communication, request contact information, forward, collect, report, send messages, or change candidate status unless the user gives the next explicit step.

## Flow 5A: Long Resume Segmented Extraction

Use this when Flow 5 cannot copy the full online resume or the resume is longer than the visible panel.

1. Try full copy once only. If clipboard text is empty, stale, clearly truncated, or contains mostly contact-list text, switch to segmented extraction.
2. Keep Chrome maximized. If using screenshots for extraction, start at normal browser zoom, then try 90 percent zoom as the default long-resume setting. Use 80 percent only after a validation screenshot confirms Chinese text is still readable. Avoid smaller zoom levels for resume extraction unless the user explicitly accepts lower confidence.
3. After changing browser zoom, capture one validation screenshot before batch extraction. If text is visibly blurry, restore a larger zoom rather than trying to infer unclear details.
4. Keep the online resume panel open and derive the current resume body area from the latest window screenshot. Do not assume coordinates from a previous screen size.
5. Focus the resume content area by clicking a safe blank margin in the resume body, not the chat list, company name, magnifier icon, right-side action panel, or visible business buttons. If a safe blank area is unavailable, skip the click and use a verified scroll method.
6. Prefer screenshots cropped to the resume body panel so sidebars, popups, and the underlying chat do not dilute OCR or visual reading.
7. Capture or read only the visible resume segment, then immediately extract facts into the structured summary fields. Do not keep raw segment text after extracting facts.
8. Scroll the resume content panel down by about 60 to 70 percent of the visible panel height, leaving overlap for continuity.
9. Before wheel scrolling, explicitly release modifier keys such as Ctrl. Prefer PageDown first when focus is verified; if PageDown does not move the content, use plain wheel scrolling or the resume panel scrollbar after verifying the cursor is over the correct panel. Ctrl plus wheel can accidentally change Chrome zoom and make screenshots unreadable.
10. If using the scrollbar, first move the cursor to the scrollbar thumb, then drag. If text becomes selected, press Esc, revalidate the resume view, and retry with a safer point.
11. Repeat visible extraction and fact merging until the bottom of the resume is reached, the scroll position stops changing, or the page shows the resume privacy footer/end section.
12. Deduplicate repeated headers, sidebars, candidate name blocks, platform notices, and overlapping lines between adjacent segments.
13. If a company information card or other non-business overlay opens accidentally, close it, confirm the resume view is back, and continue only if no message, status, purchase, or contact action occurred.
14. If a segment cannot be read confidently, mark that section as partially extracted instead of guessing.
15. Clear temporary screenshots and clipboard content after the run when practical. Restore the user's browser zoom to a readable state when the workflow changed it.

## Flow 6: Move To Next Contact And Repeat Resume Extraction

Use this flow after Flow 5 has extracted one contact's online resume and the user asks to process the next contact.

1. Close the current online resume overlay or panel.
2. Confirm the candidate communication list is visible again.
3. Click the next visible contact below the current contact in the unread/contact list.
4. Wait for the conversation panel to switch to that contact.
5. Repeat Flow 5 for the newly selected contact.
6. Extract only a structured hiring summary unless the user explicitly asks for more.
7. Do not send messages, click Continue Communication, request contact details, mark candidate state, or perform batch processing unless the user explicitly requests that next workflow.

## Flow 7: Process A Bounded Batch Of Unread Contacts

Use this flow when the user asks to process unread contacts in a batch.

1. Default batch size is 10 contacts. If the user specifies another count, use that count.
2. Start from the Unread filter in the communication interface.
3. Process one visible unread contact at a time.
4. For each contact, open the contact, run Flow 5, output only a structured hiring summary, then close the resume and move to the next visible unread contact.
5. Track non-sensitive visible state within the run, such as list position, name/role/date tuple, or processed count, to avoid processing the same visible contact twice.
6. If fewer than the requested number of unread contacts are available, process until the visible unread list is exhausted.
7. When the visible list is exhausted, scroll once and inspect again. Stop if no new unread contact appears.
8. Stop immediately when the page state is unknown, a captcha/risk prompt appears, a resume cannot be opened, or any action would send, publish, delete, purchase, request contact details, or change candidate state.
9. If the user provides a JD or screening criteria, include JD match notes in each summary. Do not compute a final score unless the user asks for a scoring rubric or gives one.

## Related Skills

- Use `desktop-control` for operating the existing Chrome window.
- Use `desktop-app-launcher` if Chrome cannot be found or focused reliably.
- Use `browser-automation` only for general automation judgment; do not switch this workflow to isolated Playwright unless the user explicitly changes the requirement.
