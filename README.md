# Practical Worksheet: iOS Tab Bar Controllers, Shared Data Models, Persistence, and Working with the Web

Understand how to work with tab bar controllers in iOS, share data across scenes, persist data to disk, and retrieve data from a remote API. You will explore using a structured model and handling data across multiple view controllers.

## Part 1: Setting Up the Project

### 1.1 Creating the Project

1. Open Xcode and create a new iOS App project named `RainbowTabs`.
2. Set the "Main Interface" to `Main.storyboard`.

### 1.2 Initial View Controller Setup

1. In `Main.storyboard`, select the existing **View Controller** and set its background color to **Red** using the Attributes Inspector (under View).
2. With the **View Controller** selected, go to `Editor -> Embed In -> Tab Bar Controller`. This will embed the **Red View Controller** in a **Tab Bar Controller**.

   - The **Tab Bar Controller** now manages the **Red View Controller** as one of its tabs.
   - For each view controller managed by the tab bar, a `UITabBarItem` is automatically created, associated with that view controller.

---

## Part 2: Adding More Tabs

### 2.1 Adding Another View Controller

1. Drag a new **View Controller** onto the storyboard.
2. Change its background color to **Orange** using the Attributes Inspector.
3. Control-drag from the **Tab Bar Controller** to the new **Orange View Controller** and select `Relationship Segue -> view controllers`. This adds the orange view to the tab bar’s list of view controllers.

### 2.2 Customizing Tab Bar Items

1. Select the **Red View Controller**'s tab bar item.

   - In the **Attributes Inspector**, set the **Title** to `Red`, the **Image** to `r.square`, and the **Selected Image** to `r.square.fill`.
   - Set the **Badge** value to `1`.
2. Select the **Orange View Controller**'s tab bar item.

   - In the **Attributes Inspector**, set the **Title** to `Orange`.
   - You can choose predefined icons by typing symbol names into the **Image** field (try using SF Symbols, such as `o.circle` for the **Image** and `o.circle.fill` for the **Selected Image**).
3. **Testing Layout in Landscape Mode:**

   - Test the tab bar interface in landscape mode by rotating the simulator or device to ensure the tab bar is still functional.

---

## Part 3: Programmatic Customization of Tab Bar Items

### 3.1 Badge Customization

1. Rename the default `ViewController.swift` file to `RedViewController.swift` to reflect the red tab’s functionality.
2. In `RedViewController.swift` (previously called `ViewController.swift`), insert the following line in the `viewDidLoad()` method to programmatically set a badge on the red tab:

   ```swift
   tabBarItem.badgeValue = "!"
   ```
3. To remove the badge when the view disappears, override the `viewWillDisappear(_:)` method:

   ```swift
   override func viewWillDisappear(_ animated: Bool) {
       super.viewWillDisappear(animated)
       tabBarItem.badgeValue = nil
   }
   ```

---

## Part 4: Adding More Tabs

### 4.1 Adding Additional View Controllers

1. Drag and drop additional **View Controllers** onto the storyboard and connect each one to the **Tab Bar Controller** using the relationship segue (`Relationship Segue -> view controllers`).
2. Customize the background color and tab bar item for each new view controller:
   - Use colors such as **Green**, **Blue**, and **Yellow** for variety.
   - Set titles like **Green**, **Blue**, and **Yellow**, and use SF Symbols for their images (e.g., `g.circle`, `b.circle`, etc.).

### 4.2 Handling Excess Tabs

- If more than five tabs are added, iOS automatically creates a "More" tab, which lists additional tabs in a table view. Keep in mind that adding too many tabs can clutter your app’s UI.

> **Best Practice:** Limit the number of tabs to 5 for iPhone apps and use more only if necessary for iPad apps.

---

## Part 6: Sharing Data Among Tab Bar Scenes

In this part, we’ll create a shared data model using a `UserModel` struct that is accessible by all view controllers, allowing text to be synchronized across all tabs.

### 6.1 Defining the Shared Model

1. **Create the `UserModel` struct**:
   - In Xcode, create a new Swift file named `UserModel.swift`:

     ```swift
     import Foundation

     struct UserModel: Codable {
         var text: String
     }

     class SharedDataModel {
         static let shared = SharedDataModel()
         private init() {}

         var userModel: UserModel? {
             didSet {
                 // Notify observers when the user changes
                 NotificationCenter.default.post(name: .userDidChange, object: nil)
             }
         }
     }

     extension Notification.Name {
         static let userDidChange = Notification.Name("userDidChange")
     }
     ```

### 6.2 Adding Text Fields to Each Tab

1. **Modify each ViewController (e.g., RedViewController, OrangeViewController):**

   - Add a `UITextField` to each view controller in the storyboard.
   - Create an `IBOutlet` for each text field in its corresponding Swift file.
   - For example, in `RedViewController.swift`:

     ```swift
     import UIKit

     class RedViewController: UIViewController {
         @IBOutlet weak var textField: UITextField!

         override func viewDidLoad() {
             super.viewDidLoad()

             // Set initial text from shared data
             textField.text = SharedDataModel.shared.userModel?.text

             // Register for user data changes
             NotificationCenter.default.addObserver(self, selector: #selector(updateTextField), name: .userDidChange, object: nil)
         }

         @objc func updateTextField() {
             textField.text = SharedDataModel.shared.userModel?.text
         }

         // Update shared data when the text field changes
         @IBAction func textFieldEditingChanged(_ sender: UITextField) {
             SharedDataModel.shared.userModel = UserModel(text: sender.text ?? "")
         }

         deinit {
             NotificationCenter.default.removeObserver(self)
         }
     }
     ```
2. **Repeat this setup for the other view controllers** (e.g., `OrangeViewController`, `GreenViewController`), so that the text is synchronized across all tabs.

### 6.3 Testing the Shared Data

1. **Run the App:**
   - Enter text into the text field in one tab.
   - Switch to another tab and verify that the text field is updated automatically, reflecting the shared `UserModel` data.

---

## Part 7: Persisting Data Using Codable and Serialization

### 7.1 Saving and Loading Data to Disk

1. **Modify the SharedDataModel class** to persist and load the `UserModel` object using `Codable` and serialization to disk:

   ```swift
   import Foundation

   class SharedDataModel {
       static let shared = SharedDataModel()
       private let fileURL: URL

       private init() {
           // Define the file path in the app’s documents directory
           let documentsDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
           fileURL = documentsDirectory.appendingPathComponent("userModel.json")
           loadUser()
       }

       var userModel: UserModel? {
           didSet {
               // Save user to disk when it's updated
               saveUser()
               NotificationCenter.default.post(name: .userDidChange, object: nil)
           }
       }

       private func saveUser() {
           do {
               let data = try JSONEncoder().encode(userModel)
               try data.write(to: fileURL)
               print("User data saved successfully.")
           } catch {
               print("Failed to save user data: \(error)")
           }
       }

       private func loadUser() {
           do {
               let data = try Data(contentsOf: fileURL)
               userModel = try JSONDecoder().decode(UserModel.self, from: data)
               print("User data loaded successfully.")
           } catch {
               print("Failed to load user data: \(error)")
           }
       }
   }
   ```
2. **Test Persistence**:

   - Run the app, input text in one of the tabs, and close the app.
   - Reopen the app and verify that the text persists across app sessions.

### 7.2 How to Close and Reopen an App

   To test persistence in your app:

- **Closing the app:** Press the home button or swipe up from the bottom (depending on your iPhone model) to exit the app.
- **Reopening the app:** Tap the app icon on the home screen to relaunch the app and verify that the persisted data is loaded.

---

## Part 8: Internet Access and Fetching Data from a Mock API

### 8.1 Setting Up a Mock API

1. **Using MockAPI or Mocky to Create a Mock API**:
   - Go to a service like [Mocky](https://mocky.io/) or [MockAPI](https://mockapi.io/) and create a simple mock API that returns dummy data for the `UserModel`. For example:

     ```json
     {
         "text": "dummy data from the web"
     }
     ```
   - Generate a URL for this API.

### 8.2 Fetching Data from the API

1. **Create a Menu Item to Fetch Data**:

   - Add a "Fetch Data" button or menu item to one of the view controllers (e.g., **RedViewController**).
2. **Perform the Network Request**:

   ```swift
   @IBAction func fetchData(_ sender: UIButton) {
       let url = URL(string: "https://your-api-url")!

       let task = URLSession.shared.dataTask(with: url) { data, response, error in
           if let data = data {
               do {
                   let userModel = try JSONDecoder().decode(UserModel.self, from: data)
                   DispatchQueue.main.async {
                       SharedDataModel.shared.userModel = userModel
                   }
               } catch {
                   print("Failed to decode JSON: \(error)")
               }
           }
       }

       task.resume()
   }
   ```
3. **Test API Integration**:

   - Run the app and select the "Fetch Data" button. Verify that the text in all tabs updates with the fetched data from the API.

---

## To Learn More

1. **Developing in Swift - Fundamentals - Unit 3, Lesson 8 - "Tab Bar Controllers"**
2. **Developing in Swift - Fundamentals - Unit 3, "Guided Project: Personality Quiz"**
3. **Developing in Swift - Data Collections - Unit 1, Lesson 7 - "Saving Data"**
4. **Developing in Swift - Data Collections - Unit 2, Lessons 4, 5, and 6 - "Working with the Web"**
5. **Developing in Swift - Data Collections - Unit 2, "Guided Project: Restaurant"**
