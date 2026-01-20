# Client Dashboard Documentation

## Overview

The Client Dashboard (or Homepage) is the primary landing page that greets clients upon login. It serves as a centralized hub for managing freelance projects, discovering top talent, browsing consultation profiles, and accessing support resources. The dashboard provides quick access to key functionalities while maintaining a clean, intuitive interface.

---

## Table of Contents

1. [Header Section](#header-section)
2. [Greetings Section](#greetings-section)
3. [Job Overview Section](#job-overview-section)
4. [Searching Mechanism](#searching-mechanism)
5. [Top Consultation Profiles Section](#top-consultation-profiles-section)
6. [Recently Viewed Profiles Section](#recently-viewed-profiles-section)
7. [Help & Resources Section](#help--resources-section)
8. [Footer Section](#footer-section)

---

## Header Section

### Purpose
The header provides navigation and quick access to core functionalities along with user account management and notifications.

### Layout & Structure

**Left Side Navigation (3 Primary Buttons)**
- **Find Freelancers**
- **Manage Work**
- **Reports**

**Right Side Icons (3 Icons)**
- **Support Icon** - Access to help and support system
- **Notification Icon** - Display notifications count badge
- **Profile Icon** - User account management

### Components Details

#### Left Side Buttons - Dropdown Menus

**1. Find Freelancers Button**
- Hover Action: Dropdown menu appears
- Menu Items:
  - **Post a Job** - Navigate to job posting form
  - **Search Freelancers** - Navigate to freelancer search/discovery page
  - **Talents You've Hired** - View previously hired freelancers
  - **Saved Profiles** - View saved/bookmarked freelancer profiles

**2. Manage Work Button**
- Hover Action: Dropdown menu appears
- Menu Items:
  - **Your Contracts** - View active and past contracts
  - **Job Biddings** - View bids received on posted jobs

**3. Reports Button**
- Hover Action: Dropdown menu appears
- Menu Items:
  - **Transaction Summary** - View payment history and financial reports
  - **Job History** - View complete history of posted jobs

#### Right Side Icons

**1. Support Icon**
- Action: Click to open support/help widget or page
- Displays support contact options, FAQs, or live chat

**2. Notification Icon**
- Behavior: Displays notification badge (red dot or count)
- Action: Click to display notification panel/dropdown
- Shows recent notifications (new bids, job completions, messages, etc.)
- Link to view all notifications

**3. Profile Icon**
- Action: Click to display dropdown menu (not hover)
- Menu Items:
  - **Your Profile** - View and edit client profile
  - **Membership Plan** - View current subscription tier and upgrade options
  - **Account Settings** - Manage security, preferences, privacy settings
  - **Log Out** - Sign out from the application

### Styling & UX

- **Background**: Clean, professional header with consistent branding
- **Spacing**: Adequate padding between elements
- **Hover States**: Clear visual feedback on hover with slight background color change
- **Active States**: Highlight active menu items to indicate current page
- **Responsive**: On mobile, collapse navigation into hamburger menu
- **Sticky**: Header remains fixed at top during page scroll

---

## Greetings Section

### Purpose
Personalized greeting to create a welcoming user experience and establish context for the current session.

### Layout & Structure

**Format**: `Good [Morning/Afternoon/Evening], [Client Name]! 👋`

- **Morning**: 6 AM - 11:59 AM
- **Afternoon**: 12 PM - 5:59 PM
- **Evening**: 6 PM - 5:59 AM

### Components

- **Dynamic Greeting Text** - Based on current time
- **Client Name** - Fetched from user profile (first name or full name)
- **Emoji/Icon** - Waving hand or greeting icon for visual appeal
- **Subtext** (Optional): Brief motivational or contextual message (e.g., "You have 2 new bids on your jobs")

### Styling

- **Font Size**: Large and prominent (H2 or H3 heading)
- **Color**: Primary brand color or dark text
- **Placement**: Top of dashboard, below header, spanning full width with padding

---

## Job Overview Section

### Purpose
Display quick summary of posted jobs and provide easy access to post new jobs. Allows clients to see at-a-glance status of their job postings.

### Layout & Structure

**Title**: "Your Jobs" or "Active Jobs"

**Two States**:

#### State 1: No Jobs Posted
- **Display**: Only a call-to-action button
- **Button**: "Post Your First Job" or "Post a Job"
- **Visual**: Large, centered card with icon and encouraging message
- **Message**: "You haven't posted any jobs yet. Start by posting your first job!"
- **Link**: Navigate to job posting form

#### State 2: Jobs Posted
- **Display**: Horizontal scrollable card layout (carousel/slider effect)
- **Card per Job**:
  - Job title
  - Brief description (first 50-100 characters)
  - Status badge (Open, In Progress, Completed, Closed)
  - Number of bids received
  - Date posted
  - Budget/Hourly rate
  - Link to view details or manage job
  - Quick action button (View Bids, Close Job, Edit Job)

- **Last Card**: "Post a New Job" button
  - Sticky at end for easy access
  - Prominent CTA design
  - Links to job posting page

### Scrolling Behavior

- **Horizontal Scroll**: Smooth scrolling with arrow buttons on left/right ends
- **Touch Support**: Swipe gesture on mobile devices
- **Indicator Dots**: Show current position in carousel (optional)
- **Auto-play**: Disabled (manual scrolling preferred)

### Card Design

- **Dimensions**: Fixed width cards (250-300px), flexible height
- **Shadow**: Subtle shadow for depth
- **Hover State**: Slight lift effect, darker shadow
- **Border**: Optional subtle border or rounded corners

### Data Requirements

**Per Job Card**:
- `jobId`
- `jobTitle`
- `jobDescription`
- `status` (Open, In Progress, Completed, Closed)
- `bidCount`
- `datePosted`
- `budget` / `hourlyRate`
- `contractType` (Fixed, Hourly)

---

## Searching Mechanism

### Purpose
Provide dual-mode search functionality to help clients find talent or discover projects. Acts as a gateway to the main search/discovery page with pre-filtered results.

### Layout & Structure

**Title**: "Find What You Need"

**Toggle Buttons** (Side by Side):
1. **Talents** - Search for individual freelancers
2. **Projects** - Search for projects (if applicable in your platform)

**Only one can be active at a time**

### Search Input

- **Placeholder Text** (Contextual):
  - If "Talents" active: "Search by skill, expertise, or name..."
  - If "Projects" active: "Search by project type, technology, or keyword..."

- **Components**:
  - Text input field
  - Search icon inside input or as button
  - Clear/Reset button (X icon)

- **Width**: Full width of container or constrained to reasonable max-width (600-800px)

### Behavior

1. **On Toggle**: 
   - Change visual state of active toggle
   - Update search placeholder
   - Clear previous search input (optional)

2. **On Search**:
   - User types keyword
   - Optional real-time suggestions (debounced)
   - On Enter or click search icon:
     - Navigate to main search page
     - Pass search query as URL parameter: `/search?mode=talents&q=keyword`
     - Pre-filter results based on selected mode

3. **Suggestion Dropdown** (Optional):
   - Show trending searches
   - Show recent searches
   - Show popular skill/project categories
   - Each suggestion is clickable and navigates to search results

### Styling

- **Section Padding**: Consistent with other dashboard sections
- **Toggle Design**: Button group or tab-like appearance
- **Input Field**: Rounded corners, subtle border, focus state with color change
- **Search Results Link**: Clear call-to-action text

---

## Top Consultation Profiles Section

### Purpose
Showcase high-quality consultation providers to encourage clients to book consultations and build relationships with experienced freelancers. Similar to Upwork's top-rated freelancers display.

### Layout & Structure

**Title**: "Top Consultation Providers" or "Expert Consultants"

**Subtitle** (Optional): "Connect with experienced freelancers for quick advice"

**Grid Layout**: 
- **Desktop**: 4-6 profiles per row, responsive
- **Tablet**: 3-4 profiles per row
- **Mobile**: 1-2 profiles per row

**Total Profiles Displayed**: 6-12 profiles (enough to scroll)

### Profile Card Components

Each consultant card displays:

1. **Profile Picture**: Circular avatar (120x120px)
2. **Name**: Full name of freelancer
3. **Specialization/Title**: "UX Designer", "Full Stack Developer", etc.
4. **Rating**: Star rating (1-5) with review count
5. **Consultation Rate**: Hourly rate for consultation
6. **Short Bio**: 1-2 line description of expertise
7. **Quick Stats** (Optional):
   - Hourly rate for consultation
   - Years of experience
   - Number of successful consultations
8. **Action Button**: "Book Consultation" or "View Profile"
9. **Availability Status**: Green dot if available now

### Card Design

- **Dimensions**: Fixed size (200-250px width)
- **Shadow**: Subtle, with hover lift effect
- **Rounded Corners**: Professional appearance
- **Hover State**: 
  - Slight lift effect
  - Button becomes more prominent
  - Background color subtle change

### End of Section CTA

**Last Card or Below**: "View All Consultants" or "Search Consultants" button
- Links to consultation-filtered search page
- Prominent design
- Links to `/search?mode=consultants` or similar

### Data Requirements

**Per Profile Card**:
- `userId`
- `profilePicture`
- `fullName`
- `specialization`
- `rating` (average)
- `reviewCount`
- `consultationRate`
- `bio`
- `yearsOfExperience`
- `isAvailableNow`
- `successfulConsultationCount`

### Filtering/Selection Logic

- **Top Profiles**: Selected based on:
  - Highest ratings
  - Most completed consultations
  - Recent activity
  - Client's relevant skill matches
- **Rotation**: Profiles can rotate daily or weekly to expose variety of consultants

---

## Recently Viewed Profiles Section

### Purpose
Provide quick access to profiles the client has previously viewed, enabling easy re-engagement with talent they were interested in.

### Layout & Structure

**Title**: "Recently Viewed Profiles"

**Conditional Display**: 
- Only show if client has viewed profiles before
- Hide if no viewing history exists

**List/Grid Format**:
- **Desktop**: 4-6 profiles in a row (similar to Top Consultants)
- **Responsive**: Adjust columns for tablet/mobile

**Maximum Display**: Last 12 viewed profiles

### Profile Card Components

Similar to Top Consultation Profiles, but includes:

1. **Profile Picture**: Circular avatar
2. **Name**: Freelancer name
3. **Specialization**: Primary skill/expertise
4. **Rating**: Star rating with review count
5. **Last Viewed**: "Viewed 2 days ago" text
6. **Action Buttons**:
   - **View Profile**: Navigate to full profile
   - **Send Message** (Optional): Quick message composer
   - **Hire**: Quick hire option

### Card Design

- Same styling as consultation profiles for consistency
- Subtle timestamp badge indicating when last viewed

### Scrolling Behavior

- **Horizontal Scroll**: Similar to job overview carousel
- **Arrow Navigation**: Left/right scroll buttons
- **Lazy Load**: Load more profiles as user scrolls

### Data Requirements

**Per Profile**:
- `userId`
- `profilePicture`
- `fullName`
- `specialization`
- `rating`
- `reviewCount`
- `lastViewedDate`
- `hourlyRate` (if applicable)

### Data Retention

- Track viewing history in database
- Retain last 50 viewed profiles per client
- Sort by most recent
- Display last 12

---

## Help & Resources Section

### Purpose
Provide easy access to common help resources, FAQs, and getting-started guides similar to Upwork's help section. Helps clients find answers quickly and reduces support burden.

### Layout & Structure

**Title**: "Help & Resources"

**Subtitle** (Optional): "Find answers and learn more about OpenGIgs"

**Card Grid Layout**:
- **Desktop**: 3-4 cards per row
- **Tablet**: 2-3 cards per row
- **Mobile**: 1 card per row

**Total Cards**: 6-8 resource categories

### Resource Categories

Suggested topics (customize based on your platform):

1. **Getting Started**
   - Title: "Getting Started with OpenGIgs"
   - Icon: Rocket or book icon
   - Description: "Learn the basics of posting jobs and hiring talent"

2. **How to Post a Job**
   - Title: "How to Post a Job"
   - Icon: Document or form icon
   - Description: "Step-by-step guide to creating your first job posting"

3. **Budget & Pricing**
   - Title: "Budget Guides & Pricing"
   - Icon: Dollar or calculator icon
   - Description: "Understand rates, fees, and how to set your budget"

4. **Payment & Billing**
   - Title: "Payment & Billing"
   - Icon: Credit card or payment icon
   - Description: "Learn about payment methods, invoicing, and refunds"

5. **Dispute Resolution**
   - Title: "Dispute Resolution"
   - Icon: Scale or chat icon
   - Description: "How we handle disputes between clients and freelancers"

6. **Security & Privacy**
   - Title: "Security & Privacy"
   - Icon: Lock or shield icon
   - Description: "Your data security and privacy on OpenGIgs"

7. **Hiring Best Practices**
   - Title: "Hiring Best Practices"
   - Icon: People or handshake icon
   - Description: "Tips for successful freelancer relationships"

8. **Contact Support**
   - Title: "Contact Support"
   - Icon: Help or headset icon
   - Description: "Get in touch with our support team"

### Card Design

- **Dimensions**: Fixed size (200-250px)
- **Icon**: Larger icon at top (48x48px)
- **Title**: Bold, clear heading
- **Description**: Subtle text explaining the resource
- **Action**: "Learn More" link or arrow icon
- **Hover State**: Subtle color change, slight lift effect

### Behavior

- **On Card Click**: 
  - Navigate to help/resources main page with the specific topic in focus
  - URL: `/help/resources?topic=getting-started`
  - Alternative: Open in modal or sidebar

- **Alternative Design** (Simpler):
  - Instead of cards, use a simple list with icons
  - Link-based approach for minimal design

### Data Requirements

**Per Resource**:
- `resourceId`
- `title`
- `description`
- `icon`
- `category`
- `link` / `slug`
- `displayOrder`

---

## Footer Section

### Purpose
Provide additional navigation, legal links, social media presence, and company information.

### Layout & Structure

**Footer Sections** (Multi-column layout):

**Column 1: About**
- Company name/logo
- Brief description of OpenGIgs
- Social media links (LinkedIn, Twitter, Facebook, etc.)

**Column 2: For Clients**
- How it works
- Browse Freelancers
- Browse Projects
- Budget Calculator
- How to Post a Job

**Column 3: Community**
- Stories & Case Studies
- Blog
- Webinars
- Community Guidelines
- Status Page

**Column 4: Support & Legal**
- Help & FAQ
- Contact Support
- Privacy Policy
- Terms of Service
- Cookie Settings
- Security

**Column 5: More**
- About Us
- Careers
- Press
- Brand Resources
- Partner Program

**Bottom Bar**:
- Copyright notice
- Year (dynamic)
- All rights reserved
- Links to Terms and Privacy

### Styling & UX

- **Background**: Darker shade (dark gray, dark blue, or brand color)
- **Text Color**: Light/white for contrast
- **Spacing**: Generous padding and margins
- **Font Size**: Smaller than body text (12-14px)
- **Links**: Hover effect (underline or color change)
- **Responsive**: 
  - Desktop: Multi-column layout
  - Tablet: 2-3 columns, stacked
  - Mobile: Single column, accordion style (optional)

### Components Details

- **Logo**: Clickable, navigates to homepage
- **Social Icons**: Linked to social media profiles
- **Newsletter Signup** (Optional): Email subscription form
- **Language/Region Selector** (Optional): If supporting multiple languages

---

## Page Layout Flow

```
┌─────────────────────────────────────────┐
│         HEADER NAVIGATION               │
│ [Freelancers] [Work] [Reports] [Icons]  │
├─────────────────────────────────────────┤
│   GREETINGS                             │
│   "Good Morning, John! 👋"              │
├─────────────────────────────────────────┤
│   JOB OVERVIEW                          │
│   [Job1] [Job2] [Job3] [+ Post Job]     │
├─────────────────────────────────────────┤
│   SEARCH MECHANISM                      │
│   [Talents] [Projects] [Search Box]     │
├─────────────────────────────────────────┤
│   TOP CONSULTATION PROFILES             │
│   [Profile1] [Profile2] ... [View All]  │
├─────────────────────────────────────────┤
│   RECENTLY VIEWED PROFILES              │
│   [Profile1] [Profile2] ... [More]      │
├─────────────────────────────────────────┤
│   HELP & RESOURCES                      │
│   [Card1] [Card2] [Card3] [Card4]       │
├─────────────────────────────────────────┤
│   FOOTER                                │
│   [Links & Information]                 │
└─────────────────────────────────────────┘
```

---

## Design Considerations

### Color Scheme
- **Primary Color**: Brand primary (used for CTAs, highlights)
- **Secondary Color**: Brand secondary (accents)
- **Background**: Light/white for main content
- **Text**: Dark gray (#333, #444) for readability
- **Borders**: Light gray (#E0E0E0, #F0F0F0)
- **Status Badges**: Green (open), Blue (in progress), Gray (closed), Red (issues)

### Typography
- **Headings**: Bold, clear hierarchy (H1 > H2 > H3)
- **Body Text**: 14-16px, line-height 1.5 for readability
- **Links**: Primary color, underlined on hover
- **Buttons**: Medium weight, clear contrast

### Spacing & Layout
- **Container Max-width**: 1200-1400px
- **Section Padding**: 40-60px vertical, 20-30px horizontal
- **Card Spacing**: 16-20px gap between cards
- **Element Spacing**: Consistent padding (8px, 16px, 24px, 32px)

### Responsive Design

**Breakpoints**:
- Mobile: 320px - 767px
- Tablet: 768px - 1023px
- Desktop: 1024px+

**Mobile Optimizations**:
- Stack sections vertically
- Full-width cards (not carousels)
- Hamburger menu for navigation
- Touch-friendly button sizes (44x44px minimum)
- Simplified layouts

---

## Interactions & Animations

### Smooth Transitions
- Hover effects: 200-300ms duration
- Page transitions: Fade or slide (200-400ms)
- Loading states: Skeleton screens or spinners

### Micro-interactions
- Button hover/active states
- Notification badge animations
- Dropdown menu slides
- Card lift on hover
- Scroll feedback

### Loading States
- Skeleton screens for cards
- Subtle spinner for search results
- Progressive content loading

---

## Performance Considerations

- **Lazy Loading**: Load profiles and resources as user scrolls
- **Image Optimization**: Use compressed images, WebP format
- **Caching**: Cache frequently accessed data (user profile, job list)
- **API Calls**: Batch requests where possible
- **Code Splitting**: Load components on demand

---

## Accessibility (A11y)

- **ARIA Labels**: For icons and buttons
- **Keyboard Navigation**: Tab through all interactive elements
- **Color Contrast**: WCAG AA compliance minimum
- **Alt Text**: For all images
- **Focus States**: Clear visual focus indicators
- **Screen Reader Support**: Proper semantic HTML

---

## API/Data Endpoints

(To be defined based on backend architecture)

### Required Endpoints

- `GET /api/client/profile` - Fetch client profile for greeting
- `GET /api/client/jobs` - Fetch posted jobs
- `GET /api/search/suggestions` - Get search suggestions
- `GET /api/profiles/top-consultants` - Fetch top consultation profiles
- `GET /api/client/recently-viewed-profiles` - Fetch recently viewed profiles
- `POST /api/client/notifications` - Fetch client notifications
- `GET /api/help/resources` - Fetch help resources

---

## Future Enhancements

- AI-powered profile recommendations
- Personalized job suggestions based on past hires
- Analytics dashboard on homepage
- Quick bid notifications with inline actions
- Project templates for quick job posting
- Saved job templates for repeated job types
- Calendar integration for scheduling consultations
- Messaging center with unread count badges

---

## Version History

- **v1.0** - Initial client dashboard documentation
- Design Date: January 2026
- Status: Draft
