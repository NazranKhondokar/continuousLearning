## **Make a text to UML diagram using `plantUML`**
Here’s a step-by-step guide to convert the provided text to a PlantUML diagram, view it, and export it as an image:

Suppose you have a description of an application settings page like:
```
Settings
1. Profile and Account Information
Step 1: Display user profile information
Step 2: Show the user's achievements.
Step 3: Allow users to update profile details:
Step 4: Update profile image (choose a predefined avatar or upload a photo).
Step 5: Edit name.
 2. Subscription Management
Step 1: Allow users to change or manage subscription plans.
 3. Appearance and Preferences
Step 1: Select app theme: Light, Dark, or System (syncs with device theme).
Step 2: Set language preferences.
Step 3: Manage currency preferences.
4. Security Settings
Step 1: Allow users to change their password.
5. Information and Support
Step 1: Access the Privacy Policy.
Step 2: Rate the app and provide feedback.
Step 3: Contact support.
```

#### **1. Convert text to PlantUML code using `ChatGPT`**
- Paste the text to **ChatGPT** and add notes like `generate plantUML code use case diagram with DB`.
- Then it will be output like
```
@startuml
left to right direction

actor "User" as user
database DB as "Database"
database Server as "GCP"

package "Settings" {
  usecase "View Profile Information" as U1
  usecase "View Achievements" as U2
  usecase "Update Profile Details" as U3
  usecase "Update Profile Image" as U4
  usecase "Edit Name" as U5
  usecase "Manage Subscription Plans" as U6
  usecase "Select App Theme" as U7
  usecase "Set Language Preferences" as U8
  usecase "Set Currency Preferences" as U9
  usecase "Set Preferred Exchange" as U10
  usecase "Change Password" as U11
  usecase "Access Privacy Policy" as U12
  usecase "Rate App" as U13
  usecase "Provide Feedback" as U14
  usecase "Contact Support" as U15
}

user --> U1
user --> U2
user --> U3
U3 --> U4 : "Change Image"
U3 --> U5 : "Edit Name"
U4 --> Server : "Save Avatar/Photo"
U5 --> DB : "Update Name"

user --> U6
U6 --> DB : "Update Subscription"

user --> U7
U7 --> DB : "Save Theme Preference"

user --> U8
U8 --> DB : "Save Language Preference"

user --> U9
U9 --> DB : "Save Currency Preference"

user --> U10
U10 --> DB : "Save Exchange Preference"

user --> U11
U11 --> Server : "Update Password"

user --> U12
user --> U13
user --> U14
user --> U15

@enduml
```

#### **2. Install PlantUML**
- Ensure you have **Java** installed (PlantUML requires it).
- Download **PlantUML** or use an IDE/editor plugin (e.g., VSCode, IntelliJ IDEA, or Eclipse) with PlantUML support.
- Alternatively, use an online PlantUML editor like [PlantText](https://planttext.com).


#### **3. Create a `.puml` File**
1. Copy the provided PlantUML code and save it into a file with the `.puml` extension. 
   - Example: `settings_usecase.puml`.


#### **4. Use PlantText for Online Viewing and Exporting**
1. Open [PlantText](https://planttext.com).
2. Paste the provided PlantUML code into the editor.
3. Click the **"Refresh"** or **"Generate Diagram"** button to view the diagram.
4. To export as an image:
   - Click **"Download as PNG/SVG"** in the output pane.

#### **5. Using IDE Plugins**
- **In VSCode**:
  1. Install the [PlantUML extension](https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml).
  2. Open the `.puml` file and press `Alt+D` or `Ctrl+P` and type `PlantUML Preview` to render the diagram.
  3. Right-click the preview and select **Export as PNG/SVG**.

- **In IntelliJ IDEA/Eclipse**:
  1. Install the PlantUML plugin via the marketplace.
  2. Open the `.puml` file, and use the built-in preview to render and export.
