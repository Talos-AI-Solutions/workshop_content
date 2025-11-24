# Poker Tournament Management System - UI/UX Design Specification

## 1. Design System Foundation

### 1.1 Color Palette

**Primary Colors**
- Primary Blue: #2563EB (Main actions, links, primary buttons)
- Primary Hover: #1D4ED8 (Hover state for primary)
- Primary Light: #DBEAFE (Backgrounds, subtle highlights)
- Primary Dark: #1E40AF (Text on light backgrounds)

**Secondary Colors**
- Success Green: #10B981 (Success states, positive actions)
- Warning Amber: #F59E0B (Warnings, cautionary states)
- Error Red: #EF4444 (Errors, destructive actions)
- Info Cyan: #06B6D4 (Informational messages)

**Neutral Colors**
- Gray 50: #F9FAFB (Page background)
- Gray 100: #F3F4F6 (Card background, subtle borders)
- Gray 200: #E5E7EB (Borders, dividers)
- Gray 300: #D1D5DB (Disabled states)
- Gray 400: #9CA3AF (Placeholders)
- Gray 500: #6B7280 (Secondary text)
- Gray 600: #4B5563 (Body text)
- Gray 700: #374151 (Headings)
- Gray 800: #1F2937 (Dark headings)
- Gray 900: #111827 (Primary text, high emphasis)
- White: #FFFFFF

**Semantic Colors**
- Gold (1st Place): #F59E0B
- Silver (2nd Place): #94A3B8
- Bronze (3rd Place): #92400E

### 1.2 Typography

**Font Family**
- Primary: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif
- Monospace: 'JetBrains Mono', 'Courier New', monospace (for numbers, IDs)

**Type Scale**
- Display: 48px / 56px line-height, weight 700 (Page titles, hero sections)
- H1: 36px / 44px, weight 700 (Main page headings)
- H2: 30px / 38px, weight 600 (Section headings)
- H3: 24px / 32px, weight 600 (Card headings, subsections)
- H4: 20px / 28px, weight 600 (Small headings)
- H5: 18px / 26px, weight 600 (List headings)
- Body Large: 18px / 28px, weight 400 (Lead paragraphs)
- Body: 16px / 24px, weight 400 (Default body text)
- Body Small: 14px / 20px, weight 400 (Secondary text, captions)
- Caption: 12px / 16px, weight 400 (Labels, helper text)

### 1.3 Spacing System

**Base Unit: 4px**
- xs: 4px (0.25rem)
- sm: 8px (0.5rem)
- md: 12px (0.75rem)
- base: 16px (1rem)
- lg: 24px (1.5rem)
- xl: 32px (2rem)
- 2xl: 48px (3rem)
- 3xl: 64px (4rem)
- 4xl: 96px (6rem)

### 1.4 Border Radius
- sm: 4px (Small elements, badges)
- base: 6px (Buttons, inputs)
- md: 8px (Cards, containers)
- lg: 12px (Modals, large cards)
- full: 9999px (Pills, circular avatars)

### 1.5 Shadows
- sm: 0 1px 2px rgba(0, 0, 0, 0.05)
- base: 0 1px 3px rgba(0, 0, 0, 0.1), 0 1px 2px rgba(0, 0, 0, 0.06)
- md: 0 4px 6px rgba(0, 0, 0, 0.07), 0 2px 4px rgba(0, 0, 0, 0.05)
- lg: 0 10px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05)
- xl: 0 20px 25px rgba(0, 0, 0, 0.1), 0 10px 10px rgba(0, 0, 0, 0.04)

---

## 2. Component Library

### 2.1 Buttons

**Primary Button**

Visual Specs:
- Background: Primary Blue (#2563EB)
- Text: White, 14px, weight 500
- Padding: 10px 20px (vertical, horizontal)
- Border radius: 6px
- Height: 40px
- Shadow: base shadow

States:
- Default: Background #2563EB
- Hover: Background #1D4ED8, shadow md
- Active: Background #1E40AF, shadow sm
- Disabled: Background Gray 300, text Gray 500, no shadow, cursor not-allowed
- Loading: Show spinner, disable interaction, opacity 0.7

Layout:
- [Icon (optional)] [Text] [Right Icon (optional)]
- Icon size: 16px, margin-right 8px

**Secondary Button**

Visual Specs:
- Background: White
- Border: 1px solid Gray 300
- Text: Gray 700, 14px, weight 500
- Padding: 10px 20px
- Border radius: 6px
- Height: 40px

States:
- Hover: Background Gray 50, border Gray 400
- Active: Background Gray 100
- Disabled: Text Gray 400, border Gray 200

**Danger Button**

Visual Specs:
- Background: Error Red (#EF4444)
- Text: White, 14px, weight 500
- Other specs same as Primary Button

States:
- Hover: Background #DC2626
- Active: Background #B91C1C

**Text Button**

Visual Specs:
- Background: Transparent
- Text: Primary Blue, 14px, weight 500
- Padding: 8px 12px
- No border, no shadow

States:
- Hover: Background Primary Light (10% opacity)
- Active: Background Primary Light (20% opacity)

**Button Sizes**
- Small: Height 32px, padding 6px 16px, text 13px
- Medium: Height 40px, padding 10px 20px, text 14px (default)
- Large: Height 48px, padding 12px 24px, text 16px

### 2.2 Form Inputs

**Text Input**

Visual Specs:
- Background: White
- Border: 1px solid Gray 300
- Border radius: 6px
- Height: 40px
- Padding: 10px 12px
- Font: 14px, Gray 900
- Placeholder: Gray 400

States:
- Default: Border Gray 300
- Hover: Border Gray 400
- Focus: Border Primary Blue (2px), shadow: 0 0 0 3px rgba(37, 99, 235, 0.1)
- Error: Border Error Red, shadow: 0 0 0 3px rgba(239, 68, 68, 0.1)
- Disabled: Background Gray 50, text Gray 500, cursor not-allowed

Layout:
- [Label]
- [Input Field]
- [Helper Text / Error Message]

**Input with Icon**

Layout:
- Left icon: Icon inside input, padding-left 36px
- Right icon: Icon inside input, padding-right 36px
- Icon size: 16px, color Gray 400
- Icon position: Absolute, centered vertically, 12px from edge

**Select Dropdown**

Visual Specs:
- Same as text input
- Right icon: Chevron down, 16px, Gray 500
- Dropdown menu appears below input

Dropdown Menu:
- Background: White
- Border: 1px solid Gray 200
- Border radius: 6px
- Shadow: lg
- Max height: 300px (scrollable)
- Options: Padding 10px 12px, hover background Gray 50
- Selected option: Background Primary Light, text Primary Blue

**Date Picker**

Visual Specs:
- Same as text input
- Right icon: Calendar icon
- Opens calendar popover on click

Calendar Popover:
- Background: White
- Border radius: 8px
- Shadow: xl
- Month/year selector at top
- Grid of dates (7 columns)
- Today highlighted with Primary Light background
- Selected date: Primary Blue background, white text
- Dates: 32px x 32px, border-radius full on hover

**Textarea**

Visual Specs:
- Same as text input
- Min height: 80px
- Resizable: vertical only
- Padding: 10px 12px

**Checkbox**

Visual Specs:
- Size: 16px x 16px
- Border: 2px solid Gray 300
- Border radius: 4px
- Background: White

States:
- Checked: Background Primary Blue, white checkmark icon
- Hover: Border Gray 400
- Focus: Shadow ring (3px, Primary Blue 10% opacity)
- Disabled: Background Gray 100, border Gray 200

Layout with Label:
- [Checkbox] [Label Text (14px, Gray 700)]
- Gap: 8px

**Radio Button**

Visual Specs:
- Size: 16px x 16px
- Border: 2px solid Gray 300
- Border radius: full (circle)
- Background: White

States:
- Selected: Border Primary Blue (6px width appears as filled circle)
- Hover: Border Gray 400
- Focus: Shadow ring

**Toggle Switch**

Visual Specs:
- Width: 44px, Height: 24px
- Border radius: full
- Background (off): Gray 200
- Background (on): Primary Blue
- Thumb: 20px circle, white, 2px margin

Animation:
- Thumb slides from left to right (0.2s ease)
- Color transition (0.2s ease)

### 2.3 Data Table

**Table Structure**

Layout:
- Container: White background, border radius 8px, shadow sm
- Header: Background Gray 50, border-bottom 1px Gray 200
- Rows: Border-bottom 1px Gray 100
- Hover row: Background Gray 50

Header Cell:
- Padding: 12px 16px
- Text: 12px, weight 600, uppercase, Gray 600, letter-spacing 0.5px
- Sortable headers: Show sort icon on hover, cursor pointer
- Sort icons: Up/down arrows, 12px, Gray 400

Body Cell:
- Padding: 16px 16px
- Text: 14px, Gray 900
- Minimum width: 100px
- Align numbers: right

Actions Column:
- Width: 120px
- Content: Icon buttons (Edit, Delete, View)
- Icons: 16px, Gray 500, hover Gray 700
- Layout: Flex, gap 8px, justify-content flex-end

**Pagination**

Layout (Bottom of table):
- Container: Padding 16px, border-top 1px Gray 200
- Left side: "Showing X-Y of Z results" (14px, Gray 600)
- Right side: [Previous] [1] [2] [3] ... [10] [Next]

Page Buttons:
- Size: 32px x 32px
- Border radius: 6px
- Text: 14px, weight 500
- Default: Text Gray 700, background transparent
- Hover: Background Gray 100
- Active page: Background Primary Blue, text White
- Disabled: Text Gray 400, cursor not-allowed
- Previous/Next: Text buttons with icons

**Empty State**

Layout:
- Center-aligned content
- Icon: 48px, Gray 400 (appropriate icon for context)
- Heading: "No players found" (18px, weight 600, Gray 900)
- Description: "Get started by adding your first player" (14px, Gray 600)
- Action button: Primary button "Add Player"
- Spacing: Icon → 16px → Heading → 8px → Description → 24px → Button

**Loading State**

Options:
1. Skeleton screens (preferred)
   - Animated shimmer effect (light gray gradient)
   - Matches table row structure
   - Shows 5-10 skeleton rows

2. Spinner overlay
   - Semi-transparent white overlay (opacity 0.6)
   - Centered spinner (32px, Primary Blue)

### 2.4 Cards

**Standard Card**

Visual Specs:
- Background: White
- Border: 1px solid Gray 200
- Border radius: 8px
- Shadow: sm
- Padding: 24px

States:
- Hover (if interactive): Shadow md, border Gray 300
- Focus (if interactive): Border Primary Blue, shadow ring

**Stat Card**

Layout:
- Same as standard card
- Icon: 40px, colored background (Primary Light), Primary Blue icon
- Label: 12px, uppercase, Gray 600, weight 600
- Value: 30px, weight 700, Gray 900
- Change indicator: 14px, Success Green or Error Red, with arrow icon

Structure:
- [Icon]          [Value]
-                 [Label]
-                 [Change: +12% from last month]

**Player/Tournament Card**

Layout:
- Same as standard card
- Avatar/Icon at top (48px circle, Primary Light background)
- Name: 18px, weight 600, Gray 900
- Metadata: 14px, Gray 600
- Stats grid: 2 columns
- Action buttons at bottom

Structure:
- [Avatar]
- [Name]
- [Email or Date]
- [Divider]
- [Stat] [Stat]
- [Stat] [Stat]
- [Divider]
- [View Details Button]

### 2.5 Modals

**Modal Overlay**

Visual Specs:
- Background: rgba(0, 0, 0, 0.5) (semi-transparent black)
- Full viewport width/height
- Backdrop blur: 2px (optional)
- Click overlay to close

**Modal Container**

Visual Specs:
- Background: White
- Border radius: 12px
- Shadow: xl
- Max width: 500px (small), 700px (medium), 900px (large)
- Margin: 48px auto
- Animation: Fade in + scale up (0.2s ease-out)

Structure:
- [Header]
- [Content]
- [Footer]

**Modal Header**

Layout:
- Padding: 24px 24px 16px 24px
- Border-bottom: 1px solid Gray 200
- Title: 20px, weight 600, Gray 900
- Close button: Right-aligned, 24px icon button, Gray 500

**Modal Content**

Layout:
- Padding: 24px
- Max height: calc(100vh - 200px)
- Overflow: auto (scrollable if needed)

**Modal Footer**

Layout:
- Padding: 16px 24px 24px 24px
- Border-top: 1px solid Gray 200
- Buttons: Right-aligned, gap 12px
- Order: [Cancel/Secondary] [Primary Action]

**Confirmation Modal**

Structure:
- Icon: 48px warning icon (Amber or Red)
- Title: "Are you sure?" (20px, weight 600)
- Description: Explain consequences (14px, Gray 600)
- Impact detail: "This will delete 12 tournament entries" (14px, Gray 700, bold)
- Buttons: [Cancel] [Delete/Confirm]

### 2.6 Toast Notifications

**Toast Container**

Visual Specs:
- Position: Fixed, top-right (16px from edges)
- Max width: 400px
- Z-index: 9999
- Stack: Multiple toasts stack vertically, 8px gap
- Animation: Slide in from right + fade in

**Toast Styles**

Success:
- Background: Success Green
- Icon: Checkmark circle, White
- Text: White

Error:
- Background: Error Red
- Icon: X circle, White
- Text: White

Warning:
- Background: Warning Amber
- Icon: Exclamation triangle, White
- Text: White

Info:
- Background: Info Cyan
- Icon: Info circle, White
- Text: White

**Toast Structure**

Layout:
- Padding: 16px
- Border radius: 8px
- Shadow: lg
- Display: flex, align-items center, gap 12px

Components:
- Icon: 20px, left side
- Content: flex-grow
  - Title: 14px, weight 600
  - Message: 13px, weight 400 (optional)
- Close button: 16px icon, right side, hover opacity 0.8

Auto-dismiss:
- Default: 5 seconds
- Error: 7 seconds
- Can be dismissed manually

### 2.7 Badges

**Badge Styles**

Visual Specs:
- Padding: 4px 12px
- Border radius: full (pill shape)
- Font: 12px, weight 600, uppercase, letter-spacing 0.3px
- Height: 24px
- Display: inline-flex, align-items center

Variants:

Primary:
- Background: Primary Light
- Text: Primary Dark

Success:
- Background: #D1FAE5
- Text: #065F46

Warning:
- Background: #FEF3C7
- Text: #92400E

Error:
- Background: #FEE2E2
- Text: #991B1B

Gray:
- Background: Gray 100
- Text: Gray 700

**Placement Badges**

1st Place:
- Background: #FEF3C7
- Text: #92400E
- Icon: Trophy, Gold

2nd Place:
- Background: #F1F5F9
- Text: #475569
- Icon: Trophy, Silver

3rd Place:
- Background: #FED7AA
- Text: #92400E
- Icon: Trophy, Bronze

### 2.8 Icons

**Icon Library**
- Use: Heroicons, Feather Icons, or Lucide
- Style: Outline for most cases, Solid for filled states

**Common Icons**
- User: Player profile
- Trophy: Tournament, placement
- Calendar: Date picker, tournament date
- Search: Search functionality
- Plus: Add new item
- Pencil: Edit
- Trash: Delete
- Eye: View details
- X: Close, cancel
- Check: Confirm, success
- Alert Triangle: Warning
- Info Circle: Information
- Chevron Down: Dropdown, expand
- Chevron Right: Navigate, next
- Filter: Filter controls
- Sort: Sorting

**Icon Sizes**
- xs: 12px (inline with small text)
- sm: 16px (buttons, inputs)
- base: 20px (default)
- lg: 24px (prominent actions)
- xl: 32px (empty states)
- 2xl: 48px (large empty states, modal icons)

---

## 3. Layout Structure

### 3.1 Application Shell

**Overall Layout**
┌─────────────────────────────────────────────┐
│  Header (Fixed)                             │
├──────────┬──────────────────────────────────┤
│          │                                  │
│ Sidebar  │  Main Content Area               │
│ (Fixed)  │  (Scrollable)                    │
│          │                                  │
│          │                                  │
└──────────┴──────────────────────────────────┘

**Header**

Specs:
- Height: 64px
- Background: White
- Border-bottom: 1px solid Gray 200
- Shadow: sm
- Position: Fixed, top 0
- Z-index: 100

Layout:
- Left: Logo + App name (flex-start)
- Center: Search bar (optional, for large screens)
- Right: User menu + notifications (flex-end)
- Padding: 0 24px

Components:
- Logo: 32px height, link to dashboard
- App name: "Poker Manager" (18px, weight 600, Gray 900)
- Search: 300px width input with magnifying glass icon
- Notifications: Bell icon button (show unread count badge)
- User menu: Avatar (32px circle) + dropdown on click

**Sidebar**

Specs:
- Width: 256px
- Background: Gray 50
- Border-right: 1px solid Gray 200
- Position: Fixed, left 0, top 64px
- Height: calc(100vh - 64px)
- Padding: 24px 16px

Navigation Items:
- Height: 40px each
- Padding: 8px 12px
- Border radius: 6px
- Gap: 4px between items
- Font: 14px, weight 500

States:
- Default: Text Gray 700, transparent background
- Hover: Background Gray 100
- Active: Background Primary Blue, text White, shadow sm

Navigation Structure:
- Dashboard (Home icon)
- Players (User icon)
- Tournaments (Trophy icon)
- Reports (Chart icon) [Future]
- Settings (Gear icon) [Future]

**Main Content Area**

Specs:
- Margin-left: 256px (sidebar width)
- Margin-top: 64px (header height)
- Padding: 32px
- Background: Gray 50
- Min-height: calc(100vh - 64px)

### 3.2 Responsive Breakpoints

**Desktop (>1024px)**
- Full sidebar visible
- Table shows all columns
- Cards in grid (3-4 columns)
- Modals: Standard widths

**Tablet (768px - 1024px)**
- Collapsible sidebar (hamburger menu)
- Table shows essential columns only
- Cards in grid (2-3 columns)
- Modals: Slightly narrower

**Mobile (<768px)**
- Hidden sidebar (opens as overlay on hamburger click)
- Table converts to card list
- Cards stack (1 column)
- Modals: Full width with 16px margin
- Bottom navigation bar (optional)

**Mobile Navigation Bar** (if implemented)
- Position: Fixed, bottom 0
- Height: 64px
- Background: White
- Shadow: lg (inverted - shadow on top)
- Items: Dashboard, Players, Tournaments, More
- Icons: 24px, active state uses Primary Blue

---

## 4. Page-Specific Designs

### 4.1 Dashboard

**Page Header**

Layout:
- Heading: "Dashboard" (H1, 36px)
- Subheading: "Overview of your poker tracking system" (16px, Gray 600)
- Date range selector: Right side (optional)
- Margin-bottom: 32px

**Stats Row**

Layout:
- Grid: 4 columns (desktop), 2 columns (tablet), 1 column (mobile)
- Gap: 24px

Stat Cards (4):
1. Total Players
   - Icon: Users (Primary Blue)
   - Value: "247"
   - Label: "Total Players"
   - Change: "+12 this month" (Green with up arrow)

2. Active Tournaments
   - Icon: Trophy (Success Green)
   - Value: "8"
   - Label: "Active Tournaments"
   - Change: "3 upcoming" (Info Cyan)

3. Total Prize Pool
   - Icon: Dollar sign (Warning Amber)
   - Value: "$125,450"
   - Label: "Total Prize Pool"
   - Change: "+$15K this month" (Green)

4. Avg Players/Tournament
   - Icon: Users group (Info Cyan)
   - Value: "32"
   - Label: "Average Players"
   - Change: "+5 from last month" (Green)

**Recent Activity Section**

Header:
- "Recent Activity" (H2, 24px)
- "View All" link (right side, text button)
- Margin-top: 48px

Content:
- White card container
- List of recent actions (last 10)
- Each item:
  - Icon (left): Colored circle with action icon
  - Content: "Player Name action Tournament Name" (14px)
  - Timestamp: "2 hours ago" (12px, Gray 500, right side)
  - Border-bottom between items

Examples:
- "John Doe placed 1st in Sunday Showdown"
- "Jane Smith registered for Friday Night Poker"
- "New tournament created: Monthly Championship"

**Quick Actions**

Layout:
- Grid: 2 columns
- Gap: 16px
- Margin-top: 32px

Action Cards:
1. Add New Player
   - Icon: User plus (Primary Blue)
   - Title: "Add New Player"
   - Description: "Register a new player"
   - Click: Opens player creation modal

2. Create Tournament
   - Icon: Trophy plus (Success Green)
   - Title: "Create Tournament"
   - Description: "Set up a new tournament"
   - Click: Opens tournament creation modal

### 4.2 Players List Page

**Page Header**

Layout:
- Left side:
  - Heading: "Players" (H1, 36px)
  - Count: "247 total players" (14px, Gray 600)
- Right side:
  - Primary button: "Add Player" (with plus icon)

**Filters & Search Bar**

Layout:
- Full width container
- Margin-top: 24px, margin-bottom: 24px
- Background: White card
- Padding: 16px

Components:
- Search input: 300px width, placeholder "Search by name or email..."
- Filter button: Secondary button, "Filters" with filter icon
- Sort dropdown: "Sort by: Name" (options: Name, Email, Date Registered, Total Tournaments)
- Results count: "Showing 50 of 247 players" (right side, Gray 600)

**Players Table**

Columns:
1. Player Name (sortable)
   - Avatar: 32px circle, initials if no photo, Primary Light background
   - Name: 14px, weight 500, Gray 900
   - Layout: Flex row, gap 12px

2. Email (sortable)
   - Text: 14px, Gray 600
   - Monospace font

3. Total Tournaments (sortable)
   - Number: 14px, Gray 900, bold
   - Aligned: Right

4. Total Winnings (sortable)
   - Currency: 14px, Gray 900, bold, monospace
   - Format: $X,XXX.XX
   - Aligned: Right

5. Registered (sortable)
   - Date: 14px, Gray 600
   - Format: "Jan 15, 2024"
   - Aligned: Left

6. Actions
   - View icon button: Opens player detail page
   - Edit icon button: Opens edit modal
   - Delete icon button: Opens confirmation modal
   - Layout: Flex row, gap 8px, right-aligned

**Mobile Card View** (<768px)

Replace table with stacked cards:
- Each player = one card
- Card layout:
  - Avatar + Name (18px, weight 600)
  - Email below name (14px, Gray 600)
  - Stats row: "X tournaments • $X,XXX won"
  - Date: "Member since Jan 2024" (12px, Gray 500)
  - Action buttons: Bottom right

### 4.3 Player Detail Page

**Back Navigation**

Layout:
- Back button: "← Back to Players" (text button with arrow)
- Margin-bottom: 24px

**Player Header Card**

Layout:
- White card, padding 32px
- Two-column layout (desktop), stacked (mobile)

Left Column:
- Avatar: 80px circle
- Name: H1 (36px, weight 700)
- Email: 16px, Gray 600, with email icon
- Joined date: "Member since Jan 15, 2024" (14px, Gray 500)

Right Column (aligned right):
- Edit button: Secondary button
- Delete button: Danger button (outline style)
- Buttons stacked: gap 12px

**Stats Overview**

Layout:
- Grid: 4 columns (desktop), 2 (tablet), 1 (mobile)
- Gap: 24px
- Margin-top: 32px

Stat Cards:
1. Total Tournaments
   - Value: "24"
   - Label: "Tournaments Played"

2. Best Placement
   - Value: "1st"
   - Label: "Best Finish"
   - Badge: Gold trophy icon

3. Total Winnings
   - Value: "$12,450"
   - Label: "Total Prize Money"

4. Win Rate
   - Value: "12.5%"
   - Label: "Top 3 Finishes"

**Tournament History Section**

Header:
- "Tournament History" (H2, 24px)
- Filter dropdown: "All Tournaments" / "Last 30 days" / "Last 3 months" / "This year"
- Margin-top: 48px

Table Columns:
1. Tournament Name
   - Text: 14px, weight 500, Primary Blue (link)
   - Date below: 12px, Gray 500

2. Date
   - Format: "Jan 15, 2024"
   - 14px, Gray 600

3. Placement
   - Badge component
   - 1st/2nd/3rd use special styling
   - Others: Gray badge with "#" prefix

4. Prize
   - Currency: 14px, weight 600
   - Green text if won, Gray if no prize
   - Aligned: Right

5. Buy-in
   - Currency: 14px, Gray 600
   - Aligned: Right

Empty State (if no tournaments):
- Icon: Trophy (Gray 400)
- Text: "No tournament history yet"
- Description: "This player hasn't participated in any tournaments"

### 4.4 Player Form Modal (Create/Edit)

**Modal Header**
- Title: "Add New Player" or "Edit Player"
- Close button (X icon)

**Modal Content**

Form Fields:

1. Name Field
   - Label: "Full Name *"
   - Input: Text input
   - Placeholder: "Enter player's full name"
   - Required indicator: Red asterisk
   - Error message: "Name is required"

2. Email Field
   - Label: "Email Address *"
   - Input: Email input with envelope icon (left)
   - Placeholder: "player@example.com"
   - Required indicator: Red asterisk
   - Validation: Real-time email format validation
   - Error messages:
     - "Email is required"
     - "Please enter a valid email address"
     - "This email is already registered"

Helper text:
- "* Required fields" (12px, Gray 500, top of form)

**Modal Footer**
- Cancel button: Secondary button
- Save button: Primary button, "Add Player" or "Save Changes"
- Save button disabled until form is valid

**Validation Behavior**
- Validate on blur for each field
- Show inline error messages below fields
- Disable submit button until valid
- On submit error (e.g., duplicate email):
  - Show toast notification (Error)
  - Highlight problematic field
  - Keep modal open

**Success Behavior**
- Close modal
- Show toast: "Player added successfully" (Success)
- Refresh players list
- Navigate to new player detail page (optional)

### 4.5 Tournaments List Page

**Page Header**

Layout:
- Left side:
  - Heading: "Tournaments" (H1, 36px)
  - Count: "45 total tournaments" (14px, Gray 600)
- Right side:
  - Primary button: "Create Tournament" (with plus icon)

**Filters Bar**

Layout:
- White card, padding 16px
- Margin-bottom: 24px

Components:
- Search input: "Search tournaments..."
- Date range filter: Two date pickers (From - To)
- Status filter: Dropdown (All, Upcoming, Completed)
- Sort dropdown: "Sort by: Date (newest first)"
- Clear filters: Text button (if filters applied)

**Tournaments Table**

Columns:
1. Tournament Name (sortable)
   - Trophy icon: 16px, Warning Amber
   - Name: 14px, weight 500, Gray 900
   - Layout: Flex, gap 8px

2. Date (sortable)
   - Format: "Jan 15, 2024"
   - 14px, Gray 600
   - Upcoming: Bold, Primary Blue
   - Past: Normal, Gray 600

3. Buy-in (sortable)
   - Currency: 14px, Gray 900
   - Monospace font
   - Aligned: Right

4. Players (sortable)
   - Number: 14px, Gray 900
   - Format: "32 players"
   - Aligned: Center

5. Prize Pool (sortable)
   - Currency: 14px, Gray 900, weight 600
   - Monospace font
   - Aligned: Right

6. Status
   - Badge component
   - Upcoming: Info Cyan badge
   - In Progress: Warning Amber badge
   - Completed: Success Green badge

7. Actions
   - View Results: Eye icon button
   - Edit: Pencil icon button
   - Delete: Trash icon button

**Status-based Styling**

Upcoming tournaments (date in future):
- Row has subtle Primary Light background tint
- Date in Primary Blue, bold

Completed tournaments:
- Normal styling

**Mobile Card View**

Each tournament card:
- Tournament name (18px, weight 600)
- Date + Status badge on same line
- Stats: "32 players • $5,000 buy-in"
- Prize pool: Large, bold (20px)
- Action buttons at bottom

### 4.6 Tournament Detail Page

**Back Navigation**
- "← Back to Tournaments"

**Tournament Header Card**

Layout:
- White card, padding 32px
- Trophy icon: 64px, Warning Amber, left side

Content:
- Tournament name: H1 (36px)
- Date: 18px, Gray 600, with calendar icon
- Status badge: Large size (if relevant)
- Edit/Delete buttons: Top right

**Tournament Info Grid**

Layout:
- Grid: 4 columns
- Gap: 24px
- Margin-top: 24px

Info Cards:
1. Buy-in
   - Value: "$500"
   - Label: "Entry Fee"

2. Total Players
   - Value: "32"
   - Label: "Participants"

3. Prize Pool
   - Value: "$16,000"
   - Label: "Total Prize Money"

4. Top Prize
   - Value: "$6,400"
   - Label: "First Place"

**Leaderboard Section**

Header:
- "Tournament Results" (H2, 24px)
- "Add Result" button: Primary button (right side)
- Margin-top: 48px

Table Columns:
1. Placement
   - Width: 80px
   - Badge with placement number
   - 1st/2nd/3rd: Special styling with trophy icon
   - Others: Gray badge

2. Player Name
   - Avatar + Name
   - 14px, weight 500
   - Link to player detail

3. Prize Amount
   - Currency: 16px, weight 600
   - Color: Success Green if won, Gray if $0
   - Aligned: Right
   - Monospace font

**Placement Styling Hierarchy**

1st Place Row:
- Background: Subtle gold tint (#FEF3C7 at 20% opacity)
- Badge: Gold background
- Trophy icon: Gold

2nd Place Row:
- Background: Subtle silver tint (#F1F5F9 at 20% opacity)
- Badge: Silver background
- Trophy icon: Silver

3rd Place Row:
- Background: Subtle bronze tint (#FED7AA at 20% opacity)
- Badge: Bronze background
- Trophy icon: Bronze

Other Placements:
- Normal white background
- Gray badge
- No icon

**Empty State** (No results yet)
- Icon: Trophy (Gray 400, 64px)
- Heading: "No results recorded yet"
- Description: "Add players and their placements to create the leaderboard"
- Action: "Add First Result" (Primary button)

### 4.7 Tournament Form Modal (Create/Edit)

**Modal Size**
- Width: 600px (medium modal)

**Modal Header**
- Title: "Create Tournament" or "Edit Tournament"
- Close button

**Modal Content**

Form Fields:

1. Tournament Name
   - Label: "Tournament Name *"
   - Input: Text input
   - Placeholder: "Enter tournament name"
   - Max length: 200 characters
   - Character counter: "0/200" (bottom right)

2. Date
   - Label: "Tournament Date *"
   - Input: Date picker
   - Default: Today's date
   - Validation: Cannot be before year 2000

3. Buy-in
   - Label: "Buy-in Amount"
   - Input: Number input with dollar icon (left)
   - Placeholder: "0.00"
   - Type: Currency (2 decimal places)
   - Optional field

4. Total Players
   - Label: "Expected Players"
   - Input: Number input
   - Placeholder: "Enter number of players"
   - Optional field
   - Helper text: "You can update this later as players register"

Form Layout:
- Two columns for Buy-in and Total Players (desktop)
- Single column for mobile

**Modal Footer**
- Cancel: Secondary button
- Save: Primary button ("Create Tournament" or "Save Changes")

### 4.8 Add Tournament Result Modal

**Modal Size**
- Width: 500px

**Modal Header**
- Title: "Add Tournament Result"
- Subtitle: "Tournament Name" (14px, Gray 600)
- Close button

**Modal Content**

Form Fields:

1. Select Player
   - Label: "Player *"
   - Input: Searchable dropdown
   - Placeholder: "Search and select player..."
   - Shows: Player name + email in dropdown
   - Required

2. Placement
   - Label: "Placement *"
   - Input: Number input
   - Placeholder: "Enter finishing position"
   - Min: 1
   - Validation: Must be positive integer
   - Required

3. Prize Amount
   - Label: "Prize Amount"
   - Input: Currency input with dollar icon
   - Placeholder: "0.00"
   - Optional
   - Helper text: "Leave empty if no prize was awarded"

**Duplicate Check**
- If player already has result in this tournament:
  - Show warning message above form (Warning Amber background)
  - Message: "This player already has a result in this tournament"
  - Disable save button

**Modal Footer**
- Cancel: Secondary button
- Save: Primary button, "Add Result"

**Success Behavior**
- Close modal
- Refresh tournament results table
- Show toast: "Result added successfully"
- New result appears in table (sorted by placement)

### 4.9 Delete Confirmation Modal

**Modal Size**
- Width: 450px

**Modal Content**

Icon:
- Warning triangle: 64px, Error Red
- Centered above text

Content:
- Heading: "Delete [Entity]?" (24px, weight 600, centered)
- Description: Explain what will be deleted (14px, Gray 600, centered)
- Impact notice: Card with Warning Amber background, rounded corners
  - Icon: Alert triangle (20px)
  - Text: "This will also delete X tournament entries"
  - Only show if there are related records

Message examples:
- Player: "Are you sure you want to delete John Doe?"
- Tournament: "Are you sure you want to delete Sunday Showdown?"

**Modal Footer**

Buttons (centered):
- Cancel: Secondary button
- Delete: Danger button, "Delete [Entity]"
- Gap: 12px

**Delete Button States**
- Default: Red background
- Hover: Darker red
- Focus: Shadow ring
- Loading: Show spinner, "Deleting..."

**Success Behavior**
- Close modal
- Show toast: "[Entity] deleted successfully"
- Navigate away from detail page (if on detail page)
- Remove from list (if on list page)

---

## 5. Interaction Patterns

### 5.1 Form Validation

**When to Validate**
- On blur: Field loses focus
- On submit: Full form validation
- Real-time: Email format, character limits

**How to Show Errors**

Invalid Field:
- Border: Error Red (2px)
- Shadow: Red ring (3px, 10% opacity)
- Error message: Below field, 13px, Error Red, with alert icon

Error Message Format:
- Icon: Alert circle (12px, Error Red)
- Text: Concise, actionable
- Examples:
  - "Email is required"
  - "Please enter a valid email address"
  - "Placement must be greater than 0"

Multiple Errors:
- Show first error only
- Re-validate as user fixes issues
- Update error message in real-time

**Success States**

Valid Field:
- Border: Success Green (2px)
- Checkmark icon: Inside input (right side)
- Only show on blur, after user input

### 5.2 Loading States

**Button Loading**
- Disable button
- Replace text with spinner + "Loading..."
- Spinner: 16px, white (for primary buttons)
- Maintain button width (prevent layout shift)

**Table Loading**
- Show skeleton rows (5-10 rows)
- Animated shimmer effect (linear gradient)
- Match table column structure
- Maintain table height

**Page Loading**
- Full page: Centered spinner (48px, Primary Blue)
- Partial: Spinner overlay on specific section

**Spinner Specs**
- Icon: Circular loading animation
- Rotation: 360deg over 1 second
- Color: Primary Blue (on light backgrounds)
- Color: White (on colored backgrounds)

### 5.3 Hover & Focus States

**Interactive Elements**

Links:
- Default: Primary Blue, no underline
- Hover: Primary Dark, underline
- Focus: Outline ring (2px, Primary Blue)

Buttons:
- Already defined in component library
- Keyboard focus: Visible focus ring (always show)

Table Rows:
- Hover: Background Gray 50
- Transition: 0.15s ease
- Cursor: pointer (if row is clickable)

Cards:
- Hover: Shadow md, border Gray 300 (if interactive)
- Transition: All 0.2s ease
- Cursor: pointer (if clickable)

Icon Buttons:
- Default: Gray 500
- Hover: Gray 700, background Gray 100 (circular)
- Active: Gray 900, background Gray 200
- Size: 32px x 32px (touch target)

### 5.4 Transitions & Animations

**General Principles**
- Duration: 0.15s - 0.3s
- Easing: ease-out (for entrances), ease-in (for exits)
- Reduce motion: Respect prefers-reduced-motion

**Modal Animations**

Enter:
- Overlay: Fade in (0.2s)
- Modal: Fade in + scale from 0.95 to 1 (0.2s ease-out)
- Stagger: Overlay then modal (50ms delay)

Exit:
- Modal: Fade out + scale to 0.95 (0.15s ease-in)
- Overlay: Fade out (0.15s)

**Toast Animations**

Enter:
- Slide in from right: Transform translateX(100%) to 0
- Fade in: Opacity 0 to 1
- Duration: 0.3s ease-out

Exit:
- Slide out to right + fade out
- Duration: 0.2s ease-in

**Dropdown Animations**

Enter:
- Slide down: Transform translateY(-10px) to 0
- Fade in: Opacity 0 to 1
- Duration: 0.15s ease-out

Exit:
- Fade out
- Duration: 0.1s ease-in

**Skeleton Loading**

Shimmer Effect:
- Gradient: Linear gradient (90deg)
- Colors: Gray 200 → Gray 50 → Gray 200
- Animation: Infinite slide from left to right
- Duration: 1.5s
- Timing: ease-in-out

### 5.5 Keyboard Navigation

**Tab Order**
- Follows logical reading order (top to bottom, left to right)
- Skip to main content link (for accessibility)
- Visible focus indicators on all interactive elements

**Keyboard Shortcuts** (Optional)
- Ctrl/Cmd + K: Open search
- Ctrl/Cmd + N: New player/tournament (context-dependent)
- Escape: Close modal/dropdown
- Enter: Submit form/confirm action
- Arrow keys: Navigate table rows

**Focus Management**

Modal Opens:
- Focus moves to first input field
- Trap focus within modal
- Escape to close

Modal Closes:
- Focus returns to trigger element

Form Submission:
- On error: Focus first invalid field
- On success: Focus confirmation message or next logical element

### 5.6 Responsive Behavior

**Sidebar**

Desktop (>1024px):
- Always visible
- Fixed position

Tablet (768-1024px):
- Collapsible
- Hamburger menu in header
- Overlay when open (semi-transparent backdrop)

Mobile (<768px):
- Hidden by default
- Hamburger menu in header
- Slides in from left as overlay
- Full height, 80% width (max 280px)

**Tables**

Desktop:
- Full table layout
- All columns visible

Tablet:
- Hide less important columns
- Priority: Name, Date, Placement/Status, Actions

Mobile:
- Convert to stacked cards
- Swipeable cards (optional)
- Pull to refresh (optional)

**Forms**

Desktop:
- Multi-column layouts where appropriate
- Side-by-side fields

Tablet:
- Mostly single column
- Some side-by-side (e.g., buy-in and total players)

Mobile:
- All single column
- Full width inputs
- Larger touch targets (48px minimum)

**Modals**

Desktop:
- Centered with margins
- Fixed width (500-900px)

Tablet:
- Slightly narrower
- More margin

Mobile:
- Full width with 16px margin
- Bottom sheet alternative (optional)
- Larger close button

---

## 6. Accessibility Guidelines

### 6.1 WCAG 2.1 Level AA Compliance

**Color Contrast**
- Text: Minimum 4.5:1 ratio
- Large text (18px+): Minimum 3:1 ratio
- UI components: Minimum 3:1 ratio
- Test all color combinations

**Focus Indicators**
- Always visible on keyboard focus
- Minimum 2px outline
- High contrast color (Primary Blue)
- Never remove outline without alternative

**Touch Targets**
- Minimum size: 44px x 44px
- Spacing: 8px between targets
- Increase button padding on mobile

### 6.2 Semantic HTML

**Structure**
- Use proper heading hierarchy (H1 → H2 → H3)
- One H1 per page
- Don't skip heading levels
- Use semantic elements: nav, main, article, aside, footer

**Forms**
- Label elements for all inputs
- Associated via for/id attributes
- Group related inputs with fieldset/legend
- Use aria-describedby for helper text

**Tables**
- Use thead, tbody for structure
- Use th for header cells with scope attribute
- Caption element for table title
- Summary for complex tables

### 6.3 ARIA Attributes

**Common Patterns**

Modal:
- role="dialog"
- aria-modal="true"
- aria-labelledby: Points to modal title
- aria-describedby: Points to description (if present)

Loading States:
- aria-busy="true" on loading containers
- aria-live="polite" for status updates
- Announce completion to screen readers

Buttons:
- aria-label for icon-only buttons
- aria-expanded for dropdowns/collapses
- aria-pressed for toggle buttons

Form Validation:
- aria-invalid="true" on invalid fields
- aria-describedby: Points to error message
- role="alert" for error messages

### 6.4 Screen Reader Support

**Text Alternatives**
- Alt text for all images (empty alt="" for decorative)
- aria-label for icon buttons ("Edit player", "Delete tournament")
- Descriptive link text (avoid "Click here")

**Status Messages**
- Announce form submission results
- Toast notifications: role="status" or role="alert"
- Loading states: Announce when complete

**Table Navigation**
- Column headers: scope="col"
- Row headers: scope="row"
- Complex tables: headers attribute

---

## 7. Performance Considerations

### 7.1 Perceived Performance

**Optimistic UI Updates**
- Update UI immediately on user action
- Revert if server request fails
- Example: Remove item from list instantly, re-add if delete fails

**Progressive Enhancement**
- Core functionality works without JavaScript
- Enhanced experience with JavaScript
- Forms submit via POST (even without JS)

**Skeleton Screens**
- Show content structure while loading
- Better perceived performance than spinners
- Maintain layout (prevent shifts)

### 7.2 Image Optimization

**Avatar Images**
- Max size: 128x128px (2x for retina)
- Format: WebP with JPEG fallback
- Lazy load off-screen images
- Placeholder: Colored circle with initials

**Icons**
- Use SVG (scalable, small file size)
- Inline critical icons
- Icon sprite for repeated icons
- System icons when possible

### 7.3 Code Splitting

**JavaScript**
- Lazy load route components
- Separate vendor bundles
- Load modals on demand
- Defer non-critical scripts

**CSS**
- Critical CSS inline
- Defer non-critical CSS
- Component-scoped styles
- Remove unused CSS

### 7.4 Data Loading

**Pagination**
- Default: 50 items per page
- Allow user to choose: 25, 50, 100
- Virtual scrolling for very long lists
- Infinite scroll (alternative to pagination)

**Caching**
- Cache API responses (5-10 minutes)
- Invalidate on mutations
- Optimistic updates
- Background refresh

---

## 8. Design Tokens Reference

### 8.1 Spacing Scale
spacing: {
xs: '4px',
sm: '8px',
md: '12px',
base: '16px',
lg: '24px',
xl: '32px',
'2xl': '48px',
'3xl': '64px',
'4xl': '96px'
}

### 8.2 Typography Scale
fontSize: {
xs: '12px',
sm: '13px',
base: '14px',
md: '16px',
lg: '18px',
xl: '20px',
'2xl': '24px',
'3xl': '30px',
'4xl': '36px',
'5xl': '48px'
}
fontWeight: {
normal: 400,
medium: 500,
semibold: 600,
bold: 700
}
lineHeight: {
tight: 1.25,
normal: 1.5,
relaxed: 1.75
}

### 8.3 Border Radius Scale
borderRadius: {
sm: '4px',
base: '6px',
md: '8px',
lg: '12px',
xl: '16px',
full: '9999px'
}

### 8.4 Shadow Scale
boxShadow: {
sm: '0 1px 2px rgba(0, 0, 0, 0.05)',
base: '0 1px 3px rgba(0, 0, 0, 0.1), 0 1px 2px rgba(0, 0, 0, 0.06)',
md: '0 4px 6px rgba(0, 0, 0, 0.07), 0 2px 4px rgba(0, 0, 0, 0.05)',
lg: '0 10px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05)',
xl: '0 20px 25px rgba(0, 0, 0, 0.1), 0 10px 10px rgba(0, 0, 0, 0.04)',
inner: 'inset 0 2px 4px rgba(0, 0, 0, 0.06)'
}

### 8.5 Z-Index Scale
zIndex: {
base: 0,
dropdown: 100,
sticky: 200,
fixed: 300,
modal: 400,
popover: 500,
tooltip: 600,
toast: 9999
}

---

## 9. Implementation Notes

### 9.1 Technology Agnostic Approach

This design can be implemented with:
- React (with Tailwind CSS, Material-UI, or custom CSS)
- Vue.js (with Vuetify, Element UI, or custom)
- Angular (with Angular Material or custom)
- Svelte (with custom components)
- Plain HTML/CSS/JavaScript

### 9.2 Component Library Options

**Use Existing Libraries**
- Tailwind UI
- shadcn/ui
- Headless UI
- Radix UI
- Material-UI
- Ant Design
- Chakra UI

**Or Build Custom**
- Follow this specification
- Use design tokens
- Build reusable components
- Document in Storybook

### 9.3 Design Handoff

**Deliverables**
- This specification document
- Figma/Sketch designs (if created)
- Design tokens (JSON file)
- Component specifications
- Interaction patterns
- Accessibility checklist

**Developer Resources**
- Style guide URL
- Component library URL
- Icon library
- Font files
- Brand assets

### 9.4 Design System Maintenance

**Version Control**
- Track design token changes
- Document component updates
- Maintain changelog

**Testing**
- Visual regression testing
- Accessibility testing
- Cross-browser testing
- Responsive testing

**Documentation**
- Component usage examples
- Do's and don'ts
- Accessibility notes
- Performance considerations

---

## 10. Future Enhancements

### 10.1 Phase 2 Features

**Advanced Filtering**
- Multi-select filters
- Save filter presets
- Advanced search operators

**Data Visualization**
- Charts for player statistics
- Tournament performance graphs
- Leaderboard visualizations

**Export Functionality**
- Export tables to CSV
- Generate PDF reports
- Print-friendly views

### 10.2 Phase 3 Features

**Dark Mode**
- Toggle in user menu
- Respect system preference
- Separate dark mode color palette

**Customization**
- User preferences
- Theme customization
- Layout options

**Internationalization**
- Multi-language support
- RTL layout support
- Currency localization
- Date format preferences

---

## Appendix A: Color Palette (Complete)

**Primary**
- 50: #EFF6FF
- 100: #DBEAFE
- 200: #BFDBFE
- 300: #93C5FD
- 400: #60A5FA
- 500: #3B82F6
- 600: #2563EB (Primary)
- 700: #1D4ED8 (Hover)
- 800: #1E40AF (Active)
- 900: #1E3A8A

**Gray**
- 50: #F9FAFB
- 100: #F3F4F6
- 200: #E5E7EB
- 300: #D1D5DB
- 400: #9CA3AF
- 500: #6B7280
- 600: #4B5563
- 700: #374151
- 800: #1F2937
- 900: #111827

**Success (Green)**
- 50: #ECFDF5
- 100: #D1FAE5
- 500: #10B981
- 700: #047857

**Warning (Amber)**
- 50: #FFFBEB
- 100: #FEF3C7
- 500: #F59E0B
- 700: #B45309

**Error (Red)**
- 50: #FEF2F2
- 100: #FEE2E2
- 500: #EF4444
- 700: #B91C1C

**Info (Cyan)**
- 50: #ECFEFF
- 100: #CFFAFE
- 500: #06B6D4
- 700: #0E7490

---

## Appendix B: Component Checklist

### Core Components
- [ ] Buttons (Primary, Secondary, Danger, Text)
- [ ] Form Inputs (Text, Email, Number, Date, Select, Textarea)
- [ ] Checkboxes & Radio Buttons
- [ ] Toggle Switches
- [ ] Data Tables with Pagination
- [ ] Cards (Standard, Stat, Entity)
- [ ] Modals (Standard, Confirmation)
- [ ] Toast Notifications
- [ ] Badges
- [ ] Icons
- [ ] Loading States (Spinners, Skeletons)
- [ ] Empty States
- [ ] Dropdowns
- [ ] Date Pickers

### Layout Components
- [ ] Header with Navigation
- [ ] Sidebar Navigation
- [ ] Page Container
- [ ] Content Grid
- [ ] Responsive Containers

### Page Components
- [ ] Dashboard Stats
- [ ] Activity Feed
- [ ] Player List Table
- [ ] Player Detail View
- [ ] Player Form
- [ ] Tournament List Table
- [ ] Tournament Detail View
- [ ] Tournament Form
- [ ] Tournament Results Table
- [ ] Add Result Form

---

## Appendix C: Responsive Breakpoints Summary

| Breakpoint | Width      | Sidebar    | Table      | Cards      | Modal      |
|------------|------------|------------|------------|------------|------------|
| Mobile     | < 768px    | Overlay    | Cards      | 1 column   | Full width |
| Tablet     | 768-1024px | Collapsible| Essential  | 2-3 cols   | Narrower   |
| Desktop    | > 1024px   | Fixed      | All cols   | 3-4 cols   | Standard   |

---

## Appendix D: Accessibility Checklist

- [ ] Color contrast meets WCAG AA standards
- [ ] Keyboard navigation works for all interactions
- [ ] Focus indicators visible on all interactive elements
- [ ] Screen reader tested (NVDA, JAWS, VoiceOver)
- [ ] All images have alt text
- [ ] Form fields have labels
- [ ] Error messages associated with fields
- [ ] Modals trap focus and announce properly
- [ ] Tables use proper semantic markup
- [ ] Skip to main content link present
- [ ] Touch targets minimum 44px x 44px
- [ ] Headings follow logical hierarchy
- [ ] ARIA attributes used appropriately
- [ ] Status messages announced to screen readers
- [ ] No content relies solely on color

---

This comprehensive UI/UX design specification provides everything needed for an implementation team or AI agent to build a professional, modern poker tournament management system. The design prioritizes usability, accessibility, and responsive behavior while maintaining a clean, professional aesthetic.