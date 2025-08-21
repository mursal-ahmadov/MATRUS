MATRUS – Management & Auditable Task Reporting Unified System

One-liner
An AI-integrated, enterprise-ready task management platform with real-time updates, LDAP auth, KPI tracking, and audit logs.


Table of Contents
1. Introduction
2. Technology stack
3. Running the application
4. Architecture Overview
5. Client application (client/)
6. Server application (server/)
7. Database
8. Docker & DevOps
9. Conclusion

Introduction
MATRUS - Management & Auditable Task Reporting Unified System is a fullstack task and project management platform inspired by tools such as Trello or Jira. The application allows teams to create boards, organize work into lists and cards, assign tasks to members, attach files, set due dates, track progress and risk, collaborate via comments and checklists, and even evaluate KPI performance for each task. Realtime collaboration is achieved through WebSockets, while the system also supports LDAP authentication for enterprise environments and provides comprehensive audit logging and notifications. A statistics dashboard summarises workload and KPI data, and the architecture is tuned for security and scalability with JWT authentication, CSRF protection, rate limiting, Redis caching and PostgreSQL at its core. The platform is composed of a Node.js/Express backend, a React (Vite) frontend and an optional admin panel, all of which can be orchestrated via Docker.

Technology stack
Backend: Node.js and Express with PostgreSQL for data storage, Redis for caching and Socket.IO for realtime updates. Libraries such as jsonwebtoken provide JWT support, ldapjs integrates with Active Directory, nodemailer is used for outbound email and multer handles file uploads. Middlewares implement security headers (Helmet), rate limiting, CSRF protection and comprehensive audit logging.

Frontend: A Vitepowered React application built with functional components. It uses MaterialUI (MUI) for the UI toolkit, react-window for virtualised lists, @hellopangea/dnd for draganddrop support, and zustand as a lightweight global store. Charts are rendered with Recharts, date handling is via datefns and dayjs, and WebSockets are consumed through socket.ioclient.

Docker & DevOps: Docker Compose defines services for the server, client, admin panel, PostgreSQL and Redis. The supplied docker-compose.yml makes it easy to spin up the full stack in development.

Database: A PostgreSQL schema is provided via database.sql; it defines tables such as boards, lists, cards, attachments, comments, activities, users, labels, statuses, notifications and many more. The dump also contains sample data to illustrate how the system records activity logs, deadlines, labels and KPI deductions.

Running the application
Docker Compose: The simplest way to get started is with Docker Compose. Running docker-compose up from the repository root will start Redis, PostgreSQL (preloaded with the schema and sample data), the backend server, the frontend client and the admin panel. The client will be served on port 5173, the API on port 5000 and the admin panel on port 3000.

Local development: To run without Docker, install dependencies in each package (npm install in server, client and admin-panel) then run npm run dev in each folder. A Redis instance and a PostgreSQL server with the provided schema must be available locally; environment variables in .env can be used to configure database credentials, JWT secrets, etc.

Architecture Overview
The project follows a classic client–server model with additional services and helpers to modularise concerns:

Server (server/) – The Express server exposes a REST API under the /api namespace. It uses a PostgreSQL connection pool defined in config/database.js and a Redis client in config/redis.js. Middlewares handle JWT authentication (middleware/auth.js), CSRF tokens (middleware/csrf.js), CORS (config/cors.js), secure headers (middleware/securityHeaders.js), request logging, rate limiting (middleware/rateLimiter.js), board permission checks (middleware/boardPermission.js), query optimisation and caching. Realtime communication is provided by Socket.IO; index.js configures the Socket.IO server and forwards events such as card creation or update to connected clients.

Client (client/) – A React SPA built with Vite. Routes are defined in src/App.jsx, mapping to pages like dashboard, board view, task list, search, notifications and statistics. UI state is managed via a global store in src/store/boardStore.js (using Zustand) and contexts such as NotificationContext. Draganddrop lists and cards are implemented with @hello-pangea/dnd, while long lists are virtualised using react-window. The client communicates with the backend via fetch and WebSocket hooks (src/hooks/useSocket.js). MaterialUI components supply a modern interface, and the theme is customised in src/theme.js.

Admin panel – Not included in this archive but referenced in the root package.json and docker-compose.yml. It likely provides log and user management dashboards.

Migrations and SQL – server/migrations/20250810_add_board_statuses.sql and other SQL files (including database.sql) define and evolve the database schema. The migration adds boardspecific status definitions.

Filebyfile analysis
The following sections describe the role of each file or group of files. They are organised by folder to aid navigation.

Top level
Client application (client/)
Configuration and entry
Shared resources
Main pages and routing (src/App.jsx)
The App.jsx file defines clientside routes using react-router-dom:
/login → Login – login page supporting LDAP and standard authentication.
/register → Register – user registration page.
/dashboard → Dashboard – lists boards accessible to the user and allows creation of new boards.
/board/:boardId → BoardPage – shows a specific board with its lists and cards. Supports draganddrop, editing titles, managing labels and statuses, scheduling deadlines, exporting boards, etc.
/my-tasks → MyTasks – displays tasks assigned to the current user, pulled from the getMyTasks API.
/search → Search – fulltext search for cards, lists and boards; results shown in SearchResults.
/notifications → NotificationsPage – lists user notifications (e.g., comments, attachments, deadline reminders) and allows marking them as read.
/statistics → StatisticsDashboard – visualises board metrics using Recharts (e.g., card counts by status, risk distribution, KPI deductions).
/ – redirects to /dashboard.

Components
The src/components folder contains dozens of reusable React components, grouped here by function:
Board management: BoardPage.jsx handles loading board data, joining the board’s Socket.IO room, rendering lists and cards via ListItem and CardItem, showing overlays like CalendarModal, CompletedCardsModal and FilterModal, managing board settings (BoardSettings and BoardPrivacySettings) and labels/statuses (LabelManagement, StatusManager), and providing toolbar actions through BoardToolbar. It also uses BoardMembers to manage membership and KPIManager to add or review KPI evaluations.
Lists and cards: AddListForm and AddCardForm allow the user to create new lists and cards. ListItem.jsx renders an individual list with its cards and includes a contextual menu (ListMenu). CardItem.jsx shows a summary of a card; clicking opens CardModal which in turn includes subcomponents such as CardDescription, CardDueDate, CardChecklist, CardAttachments, CardComments, CardAssignees, CardPriority, CardRisk, CardStatus and CardHeader. ChecklistItem.jsx manages individual checklist items within a card.
Dashboards and analytics: Dashboard.jsx lists all boards and provides quick actions. StatisticsDashboard.jsx renders charts summarising completed vs. incomplete tasks, distribution of risk levels, overdue cards and other metrics. KPIEvaluation.jsx and KPIManager.jsx allow board owners to assign KPI evaluations to users and view deductions.
User interaction: Login.jsx, Register.jsx, LDAPLogin.jsx (for domain authentication) and SimpleLDAPAdd.jsx/LDAPUserPicker.jsx/LDAPUserAutocomplete.jsx for selecting LDAP users when adding board members. EditableTitle.jsx allows inplace editing of board or card titles. Search.jsx and SearchResults.jsx implement fulltext search with debouncing. Notifications.jsx and NotificationsPage.jsx display realtime notifications. MyTasks.jsx shows tasks assigned to the loggedin user.
Utilities and modals: CalendarModal.jsx displays a calendar view of card due dates. CompletedCardsModal.jsx lists completed cards. ConfirmationDialog.jsx provides a reusable confirmation prompt. LazyImage.jsx lazily loads images. TokenRefresh.jsx transparently refreshes JWT access tokens when they expire. VirtualList.jsx and VirtualizedCardList.jsx render long lists efficiently using react-window.

Overall, the client is modular and relies on context providers, custom hooks and a global store to keep components loosely coupled. Asynchronous operations use the Fetch API with bearer tokens read from localStorage, and notifications inform the user of successes or failures.

Server application (server/)
Configuration
Middleware
Routes
Each file under server/routes defines a set of REST endpoints. Some of the most important routes are summarised below:

Services
Under server/services reside the business logic modules that power the routes. Notable services include:

Utilities
Migrations & Monitoring

Conclusion
MATRUS is a comprehensive, enterpriseready task management platform. Its backend demonstrates clean separation of concerns via services, utilities and routes and focuses heavily on security (JWT, CSRF, rate limiting, LDAP integration) and observability (audit logs, health checks, performance monitoring). The frontend provides an intuitive draganddrop interface with realtime updates, advanced features like calendar view, KPI management, domain user selection, search and filtering, and visually appealing charts. The project is containerised for ease of deployment and includes a rich set of sample data to explore its capabilities. Future enhancements could include integrating the admin panel into this repository, adding unit/integration tests for the client and exposing a public API specification (e.g., via OpenAPI).
