# Command Execution Confirmation Rule

1. **Format in Chat**: Always print the exact command(s) and their purpose in the main chat response inside a formatted ````bash ```` code snippet so that it is clearly readable with full syntax highlighting.
2. **Interactive 1-Click Confirmation**: After presenting the command in the chat, invoke `ask_question` with a short question (e.g., "آیا دستور نمایش‌داده‌شده در چت تایید است؟") and simple selectable options (e.g., "بله، اجرا شود" / "لغو / ویرایش") so the user can confirm with a single click without typing.
