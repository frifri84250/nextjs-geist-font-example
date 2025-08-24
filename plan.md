```markdown
# Implementation Plan for Skin Management Application

This plan outlines the changes needed to build a JavaScript/Next.js web application for managing user accounts and skins. The application will provide user registration/login, file upload for skins, a 3D preview using skinview3d, and CRUD operations—all managed via SQLite. Follow these steps to integrate the features into the existing codebase.

---

## 1. Database Setup and Utility Functions

### a. Create a SQLite Helper Module
- **File:** `src/lib/db.js`
- **Changes:**
  - Install SQLite packages (e.g., `sqlite3` or `better-sqlite3`).
  - Initialize a connection to a database file (e.g., `./data/skinapp.db`).
  - Create functions to run queries for user registration, login, fetching and modifying skins.
  - Set up tables:
    - **users:** `id INTEGER PRIMARY KEY, username TEXT UNIQUE, password TEXT`
    - **skins:** `id INTEGER PRIMARY KEY, user_id INTEGER, skinName TEXT, filePath TEXT, created_at DATETIME`
- **Error Handling:** Wrap DB operations in try-catch and return meaningful errors.

---

## 2. API Endpoints

Implement API routes under the new directories in the Next.js app structure. Use JSON responses and proper HTTP status codes with try-catch for error management.

### a. User Authentication
- **Registration Endpoint**
  - **File:** `src/app/api/auth/register.js`
  - **Changes:** Accept POST JSON {username, password, confirmPassword}. Validate inputs, check password equality, hash the password (using bcrypt if desired) and insert into the database.
- **Login Endpoint**
  - **File:** `src/app/api/auth/login.js`
  - **Changes:** Accept POST JSON {username, password}, verify credentials from database, set a secure cookie or session token.
  
### b. Skin Management Endpoints
- **Upload Skin**
  - **File:** `src/app/api/skins/upload.js`
  - **Changes:** Use a file upload parser (e.g., using “formidable”) to handle multipart data. Verify the user session, check the user’s current skin count (max 10), save uploaded file to `public/uploads/skins`, insert record in DB.
  - **Error Handling:** Validate file type (e.g., .png or .jpg based on expected skin format) and report if limit reached.
- **List Skins**
  - **File:** `src/app/api/skins/list.js`
  - **Changes:** Return the list of skins for the logged‑in user (limit to 10).
- **Edit Skin**
  - **File:** `src/app/api/skins/edit.js`
  - **Changes:** Accept `skinId`, and new values (new name or new file). If file is provided, replace the old file (delete it from the server) and update the DB record.
- **Delete Skin**
  - **File:** `src/app/api/skins/delete.js`
  - **Changes:** Accept `skinId`, verify ownership, remove file from disk immediately using Node.js `fs.unlink`, and delete DB record.
- **Common Considerations:** Always check for user authentication and ownership of the skin record.

---

## 3. Frontend Pages and Routes

Create pages using the Next.js (React) app directory. Each page should have a clean modern UI with consistent spacing, typography, and color.

### a. Public Pages (Login & Registration)
- **Login Page**
  - **File:** `src/app/login/page.js`
  - **Changes:** Build a form for username and password submission. On success, redirect to the dashboard.
  - **UI/UX:** Use clear input labels, a modern button style; show inline error messages.
- **Registration Page**
  - **File:** `src/app/register/page.js`
  - **Changes:** Build a similar form with extra input for confirm password. Perform client‑side validation.
  
### b. Protected Dashboard for Skin Management
- **Dashboard Page**
  - **File:** `src/app/dashboard/page.js`
  - **Changes:** After verifying the user session (e.g., via a higher‑order component or getServerSideProps), display the user’s skin collection.
  - **UI/UX:**
    - Layout the skins as cards in a grid. Each card includes:
      - A canvas element where the skinview3d instance renders the skin in 3D.
      - Three buttons: **Edit**, **Copy Link** (copies the text `/skin "direct-link"` using navigator.clipboard.writeText), and **Delete**.
    - Provide an “Upload New Skin” form at the top or in a modal.
    - Use modals and pop-up confirmations (create a standalone modal component) for deletions.
    - Ensure responsiveness with proper spacing and grid layout.

### c. Components
- **SkinCard Component**
  - **File:** `src/components/SkinCard.js`
  - **Changes:** Accept props (skin data, update/delete handlers). Inside, initialize skinview3d on the canvas once the component mounts.
- **UploadForm Component**
  - **File:** `src/components/UploadForm.js`
  - **Changes:** Allow selecting a file, entering a custom skin name, and submitting to `/api/skins/upload`.
- **Modal Component**
  - **File:** `src/components/Modal.js`
  - **Changes:** Create a generic modal for confirming deletion actions.

---

## 4. 3D Skin Viewer Integration

### a. Setup skinview3d Library
- **Approach:** Install skinview3d via npm or add its script in the dashboard page.
- **Integration:** Inside the SkinCard component, use `useEffect` to initiate skinview3d on the canvas element.
- **Error Handling:** Add fallback messages if the library fails to load.

---

## 5. Styling and UI Detailing

### a. Global Styles
- **File:** `src/app/globals.css`
- **Changes:** Define a modern color palette, typography, and spacing rules.  
- **Cards and Layout:**
  - Style the skin cards with subtle shadows, rounded corners, and ample white space.
  - Buttons should have hover and active states.
- **Images (if any):** For any placeholder images used in the planning (e.g., landing pages), use `<img src="https://placehold.co/1920x1080?text=Modern+minimalist+dashboard+interface+with+dark+theme" alt="Modern minimalist dashboard interface with dark theme" onerror="this.onerror=null;this.src='fallback.png';"/>`.

### b. Form and Modal UI
- Use clear labels, well-defined input borders, and spacing.
- The modal for deletion should overlay the dashboard with semi-transparent background, and feature a prominent confirmation message with “Cancel” and “Confirm Delete” buttons.

---

## 6. Session and Security Considerations

- Implement middleware or utility functions to check user sessions on protected API endpoints.
- Use HTTP‑only cookies or similar mechanisms to store session tokens.
- Sanitize file names and inputs to prevent directory traversal attacks.
- Validate input data both client‑side and server‑side.

---

## 7. Testing and API Validation

### a. API Endpoint Testing (using curl)
- **Registration Test:**
  ```bash
  curl -X POST http://localhost:3000/api/auth/register \
       -H "Content-Type: application/json" \
       -d '{"username": "testuser", "password": "password", "confirmPassword": "password"}'
  ```
- **Login Test:**
  ```bash
  curl -X POST http://localhost:3000/api/auth/login \
       -H "Content-Type: application/json" \
       -d '{"username": "testuser", "password": "password"}'
  ```
- **File Upload Test:**
  Use a tool like Postman or curl with form-data for file uploads.
  
### b. UI Tests
- Manually test registration, login, and dashboard flows.
- Verify that the skinview3d preview initializes correctly on each skin card.
- Ensure that the “Copy Link” feature copies the correct text.
- Check that deletion immediately updates the UI.

---

## 8. Deployment and Final Checks

- Ensure the `/public/uploads/skins` directory exists with the right write permissions.
- Update the `package.json` scripts (if necessary) to include any new dependency installations.
- Run `npm run dev` to manually verify functionality.
- Add logging mechanisms on API endpoints for troubleshooting errors in production.

---

### Summary
- A new SQLite helper (`src/lib/db.js`) is created for managing users and skins.  
- API endpoints for registration, login, skin upload, edit, list, and deletion are implemented with proper error handling.  
- Dashboard, login, and registration pages are developed with a clean modern UI incorporating skinview3d for 3D preview.  
- Reusable components (SkinCard, UploadForm, Modal) are established for maintainability.  
- Security, file validations, and session management are addressed throughout.  
- Comprehensive testing using curl and manual UI verification ensures reliable operation.
