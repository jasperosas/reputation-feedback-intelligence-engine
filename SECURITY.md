# Security review and publication checklist

**Current GitHub visibility: Private. Do not change it without explicit approval and a separate source/permission review.**

## Controls applied to the sanitized portfolio copies

- Removed historical n8n credential reference objects, saved sample execution/pin data, instance metadata and prior live spreadsheet identifiers.
- Replaced source Google Sheets document references and old live form URLs with placeholders.
- Removed literal personal email destinations; operational expressions remain but require your own accounts.
- Disabled form and spreadsheet triggers in the portfolio exports to reduce accidental execution after import.
- The original workflow topology, count of nodes and connections have been retained for educational review.

## Review scope and limitations

The audit is **static and best effort**; it does not prove that every possible secret or client-sensitive business detail has been removed, and it is not an end-to-end run. Scrub every code node, prompt and URL again before publishing. Check permissions to share course materials or client/business processes. Never commit unredacted source archives, Google Sheets with real customers, n8n credentials, tokens, or execution logs.

## Execution status

The archived analysis path connects **Gemini**. An OpenAI alternative was retained separately as an **untested recovery**, not a verified deployment. Reactivation requires an n8n instance, replacement IDs, reconnected credentials, correct sheet mapping, and end-to-end testing.

## Notes about the visual demo

The SVG and CSV in `assets/` and `examples/` are **fictional sample data**, not screenshots or recorded outputs from the workflow. Routes illustrate the archived rules and are not guaranteed runtime outcomes.
