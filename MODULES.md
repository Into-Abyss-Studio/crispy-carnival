# CashCast — Modules & Feature Reference

CashCast is a mobile-first personal finance forecasting application delivered as a single, self-contained HTML file. Although it lives in one file, it is internally organized as a clean set of modules that appear in a deliberate order: first the page markup, then the styling layer, and finally a sequence of fourteen JavaScript modules running from low-level utilities up through the user interface and application startup.

This document lists every module in the project and, within each, the complete set of features that module provides. Features are listed in the order they appear in the source so the document doubles as a map of how the application is assembled from the ground up. Everything here is described in plain language; no code or implementation detail is included.

---

## 1. Document & Markup Module

The structural skeleton of the page — the elements the rest of the application styles and brings to life.

- Page setup and metadata, including the character set, a mobile-optimized viewport that locks zoom and respects device safe areas, a theme color for the browser chrome, and the application title.
- The application shell: a centered, phone-width container that holds the whole experience.
- A sticky top bar carrying the CashCast wordmark and logo on the left and a settings button on the right.
- The main content region where the active view (dashboard or spreadsheet) is rendered.
- A calendar overlay panel: a slide-up sheet containing the month title, previous/next month controls, weekday labels, the day grid, and quick-jump controls for stepping by year or returning to today.
- A settings overlay panel: a slide-up sheet with a title, a close control, and a body region into which the settings options are rendered.
- A hidden file input used to receive a chosen backup file during import.
- A toast element used for brief, transient confirmation and error messages.
- Inclusion of the external motion library from a content-delivery network, used throughout for animation.

## 2. Styling Module

The complete visual design system that gives the application its calm, tactile, phone-native feel.

- Design tokens: the shared palette (deep backgrounds, layered surfaces, hairline borders, primary text and muted text), the semantic colors for income/positive, expense/negative, forecast, and informational accents, plus reusable values for shadows, corner radii, comfortable touch-target sizing, the maximum app width, and safe-area spacing.
- Global base styles: typography, default element resets, removal of tap highlights, suppression of scrollbars, and overscroll behavior tuned for a contained app feel.
- Application shell styling: the centered phone-width frame, content padding, and the sticky, blurred top bar with its branded logo tile and the pressable settings icon button.
- Today Ribbon styling: the ribbon surface; the distinctive active-state treatment for "today" using layered shading and subtle cross-hatching; the date button that opens the calendar; the prominent "Today" badge; the day-navigation arrow buttons; the editable Bank, In, and Out fields with their colored indicator dots and focus states; and the full-width note field.
- Forecast Ribbon styling: the four-metric layout, each metric's label, large value, and supporting caption, with positive, forecast, and negative color variants; the press state; and the expandable detail panel beneath the ribbon, including its title, explanatory text, calculation rows, totals, and inline editable rows.
- Cards and visualizations styling: section labels, the card surface, card headers, hints and subtitles, chart legends, the responsive chart wrapper, the two-up gauge grid with its single-column fallback on very narrow screens, the gauge value overlay, gauge captions and status tags, the upcoming-expenses list with date blocks and proportional bars, the cash timeline with its track, progress fill, "now" marker and expense pins, and a friendly empty-state treatment.
- Calendar and sheet styling: the dimmed overlay backdrop with its open transition, the slide-up sheet with a grab handle, sheet titles and close affordance, and the full calendar layout — month header and navigation, weekday row, day grid, individual day cells with today/selected highlighting and per-day activity dots, and the quick-jump row.
- Settings styling: grouped sections, large tappable action rows with leading icons, toggle rows, the on/off switch, the currency selector, a destructive "danger zone" treatment, and a footer line for app metadata.
- Spreadsheet view styling: the back/header bar, the toolbar with its add-row action and record count, the table with its header row, individual data rows, right-aligned editable numeric cells with income/expense coloring, the per-row note field, the delete control, and an empty-state message.
- Toast and utility styling: the floating toast bubble with show/error states, a hidden helper, and tabular-number formatting for aligned figures.

## 3. Constants Module

The fixed values shared across the whole application.

- The application version stamp and the local-storage key under which data is saved.
- The rolling window length used for the income average.
- Day-of-week names, full month names, and abbreviated month names.
- The supported currencies and their display symbols.
- A one-time check for whether the motion library loaded successfully, used to gracefully skip animation when it is unavailable.

## 4. Date Helpers Module

A small toolkit for working with calendar dates consistently, always using local-midnight dates in year-month-day form.

- Formatting a date into the standard year-month-day text and parsing that text back into a date.
- Producing today's date.
- Stepping a date forward or backward by a number of days, and by a number of months.
- Finding the first and last day of a month, and the number of days in a given month.
- Finding the start of the week (Monday-based).
- Measuring the number of days between two dates and reading the weekday of a date.
- Producing human-friendly date labels: a short "weekday, month day" form and a longer "month day, year" form.
- Testing whether a date falls within a given range.

## 5. Storage Adapter Module

The single, isolated point of contact with the browser's local storage.

- Loading saved data, tolerating missing or corrupted data by returning nothing rather than failing.
- Saving the current data as plain text, capturing the version, all records, and settings, with a graceful warning if storage is unavailable or full.

## 6. State Store Module

The central place that holds application state and notifies the interface when it changes.

- Defining the default starting state: the version, an empty set of records, default settings, and the volatile interface state (selected date, current view, which detail panel is open, and the calendar's displayed month).
- A lightweight subscription mechanism so the interface can re-render when state changes.
- A way to broadcast change notifications to all subscribers.
- A way to persist the current state through the storage adapter.
- Hydration on startup: merging any previously saved records and settings into the live state.

## 7. Domain Model & Validation Module

The definition of the core data object and the routines that keep user-entered values sane.

- The shape of a daily record: a date, an optional bank balance, income, expense, and a note, including a way to produce a fresh blank record.
- Retrieving the record for a given date, returning a blank one when none exists.
- Producing the list of stored dates in chronological order.
- Coercing free-form numeric input into a clean non-negative number, treating empty input as zero.
- Interpreting bank input specially, allowing it to be genuinely empty (unknown) rather than forced to zero.

## 8. Actions Module

The complete set of operations that change state; each mutates the data, persists it, and triggers the appropriate refresh.

- A low-level field write that updates a single field of a date's record, enforces validation, and automatically discards a record that has become completely empty, then saves.
- A field commit used by all editable inputs that writes the value and then refreshes only the derived parts of the screen, deliberately leaving the input fields and top controls intact so the keyboard stays open and a tap on another control still registers in the same gesture.
- Selecting a date, which updates the chosen day, aligns the calendar to that month, closes any open detail panel, and refreshes.
- Toggling a forecast detail panel open or closed.
- Switching between the dashboard and the spreadsheet view.
- Changing the display currency.
- Deleting a single record.
- Clearing all records at once.
- Importing data from an external object, validating it at the boundary: keeping only entries with valid dates, sanitizing each record's values, optionally adopting a recognized currency setting, and reporting how many records were imported.

## 9. Derived Selectors Module

Pure calculations computed from the stored records; these never change data, they only interpret it.

- Money formatting honoring the chosen currency, with sensible decimal handling, and a signed variant that always shows a leading plus or minus.
- Finding the most recent known bank balance on or before a given date (the forecasting anchor).
- The rolling daily income average over the recent window.
- Summing income and summing expenses across a date range.
- Week-to-date income, month-to-date income, and the month's total expenses.
- Projecting a day-by-day balance series from the anchor forward: applying actual recorded income and expenses through today, then projecting ahead using the daily income average minus known future expenses, while honoring any later real bank corrections.
- The forecasted end-of-month balance derived from that projection.
- The list of upcoming known expenses within a horizon.
- The income-pace comparison: actual income for the month versus what the recent average would predict for the elapsed days, expressed as a ratio.

## 10. Motion Helpers Module

Reusable animation behaviors that make changes feel tactile, all of which fall back to instant updates when the motion library is absent.

- Animated number transitions that count a displayed value smoothly from its previous figure to its new one.
- Expanding and collapsing panels by animating their height.
- A gentle "pop" reveal for an element.
- A staggered entrance for a group of elements.

## 11. SVG & Chart Builders Module

The hand-built, crisp vector visuals and the small geometry helpers behind them.

- A helper for constructing vector graphic elements, plus a safe measurement of a path's length and helpers for converting angles to points and describing arcs.
- The projected-balance chart: an area-and-line chart showing the actual balance as a solid line with a filled area and the forecast as a dashed line, with a zero baseline warning when balances could go negative, markers on expense days, a "today" divider and point, an endpoint marker colored by outcome, date labels, an empty state when there is too little data, and an optional draw-on animation.
- The income-pace gauge: a semicircular gauge whose fill and color reflect whether income is on, ahead of, or behind pace, with an "on pace" reference tick and an optional sweep animation.
- The end-of-month ring gauge: a circular progress ring colored by the forecast outcome, with an optional sweep animation.

## 12. DOM Helpers Module

The minimal building blocks used to assemble the interface and surface messages.

- A concise element builder that sets classes, attributes, inline markup, values, and event handlers and appends children.
- Shorthand helpers for selecting a single element or a list of elements.
- The toast notification helper that shows a brief message, styles it as normal or error, and clears it automatically.

## 13. Views & Render Module

The presentation layer that turns state into the on-screen experience; this is the largest module and the heart of the interface.

- The top-level render dispatcher that shows either the spreadsheet view or the dashboard based on the current view.
- The dashboard: the Today Ribbon with its calendar-opening date button, the always-present previous/next day arrows surrounding either the "Today" badge or a jump-to-today control, the active-state treatment when viewing today, the editable Bank, In, and Out fields, and the note field; followed by the Forecast Ribbon's four metrics; the detail-panel container; the visualization section; and the entrance animation for the ribbon and metric values.
- The visualization builder, which renders the scrollable cards and can either play full reveal animations on first paint or refresh quietly without replaying motion: the projected-balance chart with its descriptive subtitle and legend; the income-pace gauge with its value, comparison caption, and status tag; the end-of-month gauge with its value, change-versus-now caption, and a cash-runway tag estimating days until funds would run out; the upcoming-expenses list with proportional bars and human-friendly timing; and the cash timeline.
- The lightweight derived-refresh routine that updates the four forecast metrics in place, adjusts the forecast metric's color when its sign changes, rebuilds only the visualizations, and refreshes an open detail panel only when the user is not actively editing inside it.
- The runway calculation that scans the projection for the first day the balance would turn negative.
- The editable field builder for the ribbon's numeric inputs and the metric builder for the forecast tiles, plus the routine that animates a metric to its new value.
- The forecast detail panels shown beneath the ribbon, each with an explanation and, where editing is meaningful, inline editable underlying entries: a shared panel frame; a list of income entries for a range with inline editing; the week-income panel; the month-income panel; the end-of-month forecast panel that breaks the figure into anchor balance, actual movement so far, and projected movement ahead so the parts reconcile to the total; and the month-expenses panel listing each expense with inline editing and a total.
- The cash timeline visualization: a horizontal track with a progress fill, a "now" marker, and pins for upcoming expenses sized by relative impact, plus a summary line.
- The spreadsheet view: a header with a return-to-dashboard control, a toolbar with an add-row action and a record count, and a table of every record sorted newest-first with right-aligned editable bank, income, and expense cells, an editable note per row, and a delete control; cell edits here persist without rebuilding the table so editing stays fluid.
- The editable spreadsheet cell builder and the add-row flow that prompts for a new date, validates its format, creates the record, and selects it.

## 14. Calendar Module

The month-picker experience presented in a slide-up sheet.

- Opening the calendar aligned to the selected date's month and revealing the sheet.
- Closing the calendar.
- Rendering the calendar: the month-and-year title, a correctly offset Monday-based day grid, today and selected-day highlighting, small per-day dots indicating recorded income, expense, or bank values, tapping a day to select it and close, and a staggered entrance animation for the day cells.

## 15. Settings Module

The practical controls for owning, backing up, and shaping the data.

- Opening and closing the settings sheet.
- Rendering the settings content: a data group with export and import actions; a view group with the spreadsheet-view toggle and the currency selector; a danger zone with the clear-all-data action; and a footer showing the app version, the record count, and a reminder that data is stored locally.
- A reusable builder for the large settings action rows.
- Exporting all data as a downloadable, human-readable backup file stamped with the app name, version, export time, settings, and records.
- Clearing all data behind a confirmation prompt.
- Handling an imported file: reading it, parsing and validating it through the import action, and reporting success or a clear error.

## 16. Initialization Module

The startup wiring that brings everything online.

- Binding all global controls: the settings open/close behavior, dismissing overlays by tapping their backdrop, the calendar's month, year, and jump-to-today navigation, and the file-input change that begins an import.
- The startup routine that loads saved data, wires the global controls, subscribes the renderer to state changes (also keeping an open calendar or settings sheet current), and performs the first render.
- Running startup once the page is ready.

---

## Architecture at a Glance

The modules build upward in dependency order. Constants and date helpers underpin everything. The storage adapter, state store, domain model, and actions form the data layer, with all state changes flowing through explicit actions. Derived selectors interpret that data with pure calculations. The motion, vector-chart, and DOM helpers provide presentation building blocks. The views module composes those into the dashboard, detail panels, spreadsheet, and visualizations, refreshing derived content in place to keep data entry fluid. The calendar and settings modules supply the two slide-up experiences, and the initialization module wires it all together and starts the app. The result is local-first by design: the user's data stays on their device, fully owned, portable, and inspectable, with import and export as first-class features.
