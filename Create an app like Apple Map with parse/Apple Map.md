# How to make MapView similar to Apple Map using Pulley

In this tutorial we will create a simple application using Parse-Server and Mapkit.

[Parse Server](http://parseplatform.org/) is a BAAS(Backend-As-A-Service) framework and Facebook acquired it a few years ago. It is now open source. The biggest advantage is that you can easily build your server.

A brief introduction to the application features we will create.

- Create accounts and log in
- Store my current location on the parse server
- Create MapView like an Apple Map using Pulley Library

Before getting started, we will need the following:

1. Create [Back4App](https://www.back4app.com/) Account to use Parse-Server (It's free) 
2. Install CocoaPod Libraries 
   - [Parse iOS SDK](https://github.com/parse-community/Parse-SDK-iOS-OSX/)
   - [MBProgressHUD](https://github.com/jdg/MBProgressHUD)
   - [Pulley](https://github.com/52inc/Pulley)
   - [SDWebImage](https://github.com/rs/SDWebImage)

## Project Settings

I will describe the process of creating a project and installing a library and creating a Parse account.

![Create New Project](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/xcodeProject.png)

Run xcode and select the template.



![pod init](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/podInit.png)

Open Terminal and navigate to the folder where you created the project. Then run pod init.



![xcworkspace](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/xcwork.png)

When you open the project folder, you can see that the xcworkspace file has been created. Let's open this file.



```swift
target 'MyRoute' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  # Pods for MyRoute
  pod 'Parse'
  pod 'MBProgressHUD'
  pod 'SDWebImage'
  pod 'Pulley'
  
end
```

If you look at the list of files in the left panel, you will see the podfile. Please select the file and write as above.

![Pod Install](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/podInstall.png)Go back to the terminal and run the pod install command.



![LoginView](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/loginview.jpg)

![Segue](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/segue.png)

Connect LoginViewController and SignUpViewController using Segue.

![Storyboard](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/storyboard.png)

We have set the viewcontroller for login and signup.



## Parse Server Setting

We have finished installing the necessary libraries for the project. Now we will create an account for [Back4App](https://www.back4app.com/) before using Parse Server.

![Back4App](back4app.png)

Let's create an account.

![Create Parse App](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/createParseApp.png)

Let's create a Parse app.

![Parse Setting](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/clientKey.png)

When you enter the core settings menu, you can see the **applicationID** and **clientKey** information. Do not use the applicationId and clientkey shown in the picture, but use the key shown in the project you created.



![Dashboard](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/dashboard.png)

Let's create a database in advance for the app. For Parse Server, User Class is created by default. I will show you how to create a **MyPlace** Class to store a place on the server.

When you select Dashboard, click **create class** shown in the upper left corner.



![MyRoute Object](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/createMyRoute.png)

Select Custom for the class type and enter the name MyPlace.

The **MyPlace** class has the following default fields:

- objectId
- updatedAt
- createdAt
- ACL([Access Control List](https://en.wikipedia.org/wiki/Access_control_list))

`objectId` is a unique value that is created when data is stored on the server.

`createdAt` is a Date type and is saved only once when it is first created.

`updatedAt` is the same type as createdAt and records each time the data is modified.

`ACL` is used to determine whether or not a user has permission to access data.



![Add New column](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/newColumn.png)

| Field    |       Type        |
| -------- | :---------------: |
| name     |      String       |
| image    |       File        |
| location |     GeoPoint      |
| address  |      String       |
| owner    | Pointer -> PFUser |

Several fields should be added to store further information about the location. Select **Add new column** on the right and add 5 fields to **MyPlace** Class as shown in the table.

The five fields just added are described below:

`name` is the name of the current location.

`image` refer to photographs representing the current location.

`location` is stored in the current location for the GPS coordinates.

`address` records the address of the current location.

`owner` is used to store which user created MyRoute.

![Create Owner](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/owner.png)

When you add an owner field to MyRoute, you need to specify it as a User pointer, unlike the other fields.

Now it is all set. Go back to xcode and create an account and login.

## Create accounts and log in

To use the Parse Server described above, we need to create two Swift files for User and MyPlace.

![model](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/modelswift.png)

Select swift and create a new file.

```swift
import Foundation
import Parse

class User: PFUser{

    override class func query() -> PFQuery<PFObject>? {
        
        let query = PFQuery(className: "_User")
        query.cachePolicy = .networkElseCache
        return query
        
    }
}
```

Type in the User.swift file as above. We will use PFUser like NSObejct. 

```swift
import Foundation
import Parse

class MyPlace: PFObject, PFSubclassing {
    
    @NSManaged var name: NSString?
    @NSManaged var img: PFFile?
    @NSManaged var location: PFGeoPoint?
    @NSManaged var owner: PFUser?
    @NSManaged var address: NSString?
    
    static func parseClassName() -> String {
        return "MyPlace"
    }
    
    override class func query() -> PFQuery<PFObject>? {
        
        let query = PFQuery(className: MyPlace.parseClassName())
        query.cachePolicy = .networkElseCache
        return query
        
    }
    
}

```

As described above, we have added five fields for storing places in MyPlace. Parse Server has a PFUser object that manages User data and a PFObject that manages common data. MyPlace uses PFObject and PFSubclassing.



```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        
        //1. Setup Parse-Server
        let configuration = ParseClientConfiguration {
            $0.applicationId = "mmwzYDoWcVyuIhkQL50La6Q2OFMrF1Y9jeTCIzkG"
            $0.clientKey = "93tJgn4R1eh9LnsnA1DDVlUdoqviEDx6pYqXiROA"
            $0.server = "https://parseapi.back4app.com/"
            
        }
        
        Parse.initialize(with: configuration)
        PFUser.enableAutomaticUser()
        
    	//2. Setup Parse Object
        configureParseObjects()
        
        //3. Make sure you already have a login or not
        buildUserInterface()

        
        return true
    }
```

Let's set up to use Parse Server in AppDelegate.swift. First, you need to enter applicationID and clientKey to setting up Parse Server.



```swift
func configureParseObjects() {
        
        User.registerSubclass()
        MyPlace.registerSubclass()
    }
```

This is the code that registers objects to use in Parse Server.



```swift
func buildUserInterface()
    {
        
        //1. get username from UserDefaults
        let userName:String? = UserDefaults.standard.string(forKey: "username")
        
        //2. To check if you're currently signed in or not
        do {
            
            try User.current()?.fetch()
            
        } catch _ {
            return
        }
        
        //3. If the user is logged in, the login screen is skipped and moved to the MainViewcontroller.
        if((userName != nil) && (User.current() != nil) )
        {
            let mainStoryBoard:UIStoryboard=UIStoryboard(name:"Main",bundle:nil)
            
            let main : MainViewController = mainStoryBoard.instantiateViewController(withIdentifier: "MainVC") as! MainViewController
            
            window?.rootViewController = main
 
        }
        
        
    }
```

The buildUserInterface function checks whether the user is logged in. If you are logged in, go to MainViewController, otherwise go to login screen.



```swift
import Foundation
import UIKit

typealias alertHandler = (UIAlertAction) -> ()

protocol AlertMessage : class {
    
    func showAlertMessage(title: String, message: String, completion: alertHandler?)
}


extension AlertMessage where Self: UIViewController {
    
    func showAlertMessage(title: String, message: String, completion: alertHandler?) {
        
        let alert = UIAlertController(title: title, message: message, preferredStyle: .alert)
        
        alert.addAction(UIAlertAction(title: "OK", style: .default, handler: completion))
        
        self.show(alert, sender: nil)
    }
    
}

```

To reduce the amount of code we use repeatedly, we created an `AlertMessage.swift` file and created a protocol for `ViewController`.



```swift
 @IBAction func loginTapped(_ sender: Any) {
        
     //1. Ensure that the input value is not empty
        guard let username = usernameField.text, let password = passwordField.text else { return }
        
     //2. Loading 
        let spiningActivity = MBProgressHUD.showAdded(to: self.view, animated: true)
        spiningActivity.label.text = "Please wait"
        spiningActivity.detailsLabel.text = "Logging in to the server"
        
     //3. try logIn
        User.logInWithUsername(inBackground: username, password: password) { (user: PFUser?, error: Error?) in
            
            MBProgressHUD.hide(for: self.view, animated: true)
            
            
            if(user != nil)
            {
                //Remember the sign in state
                let currentUser:String? = user?.username
                
                UserDefaults.standard.set(currentUser, forKey: "username")
                UserDefaults.standard.synchronize()
                
                //Navigate to Main Page
                let appDelegate:AppDelegate=UIApplication.shared.delegate as! AppDelegate
                appDelegate.buildUserInterface()
                
            }else{
                
     self.showAlertMessage(title: "Login failed", message: error!.localizedDescription, completion: nil)
                
            }
        }
    }
```

This code is executed when the SignIn button is tapped on the LoginViewController.



```swift
 @IBAction func signupTapped(_ sender: Any) {
     
        //1. Ensure that the input value is not empty
        guard let username = usernameField.text, let password = passwordField.text, let conformPassword = confirmPasswordField.text else { return }
        
        //2. Check Password Again
        if(password != conformPassword){
            
           showAlertMessage(title: "SignUp failed", message: "Please check your password again", completion: nil)
            
            return
        }
        
        //3. create new account 
        let account:User=User()
        account.username=username
        account.password=conformPassword
        

        //4. Show Loading indicator
        let spiningActivity = MBProgressHUD.showAdded(to: self.view, animated: true)
        spiningActivity.label.text = "SignUp"
        spiningActivity.detailsLabel.text = "Please wait"
        
     	//5. try SignUp with new account
        account.signUpInBackground{(success:Bool, error:Error?)->Void in
            
            
            //6. Hide activity indicator
            spiningActivity.hide(animated: true)
            
 			var alertMessage="Registration is successful. Thank you!"
            
            if(!success)
            {
                alertMessage=error!.localizedDescription
            }

            self.showAlertMessage(title: "Welcom to MyRoute App", message: alertMessage, completion: { (action) in
                
                if(success){
                    self.dismiss(animated: true, completion: nil)
                    
                }
            })
            
        }

    }
```

This code is executed when the SignUp button is tapped on the SignUpViewController. When signup is complete, go back to LoginViewController.

![Login](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/login.gif)

Wow, We finally completed the login and SignUp functions. Now We will implement the part that stores my location and information in Parse Server, and finally We will create a MapViewController using the Pulley library.

## Saving My Places to the Parse Server

Now let's learn how to store information about places in Parse Server where I am currently. 

![FirstVC](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/firstvc.png)

FirstViewController's UI is configured as above to upload address, photo, and place names.



![Info.plist](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/infolist.png)

```
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
    <string>Shows your location on the map</string>
    
    <key>NSLocationAlwaysUsageDescription</key>
    <string>Will you allow this app to always know your location?</string>
    
    <key>NSLocationWhenInUseUsageDescription</key>
    <string>Do you allow this app to know your current location?</string>
    
    	<key>NSPhotoLibraryUsageDescription</key>
	<string>Do you allow this app to load your photo?</string>
```

We need to use CoreLocation to get your current location information from GPS. I also added permission to access the photo album.



```swift
 	@IBOutlet weak var addressLabel: UILabel!
    @IBOutlet weak var nameField: UITextField!
    @IBOutlet weak var placeImage: UIImageView!

    let locationManager = CLLocationManager()
    var currentLocation: CLLocation?
    let geocoder = CLGeocoder()
    
    let imagePickerController = UIImagePickerController()
```

The properties declared in FirstViewController are shown above. The properties related to the location information are as follows.

- `locationManager` : start and stop the delivery of location-related events to your app

- `currentLocation` : represents the location data like latitude, longitude

- `geocoder` : converting between geographic coordinates and place names

  ​

```swift
override func viewDidLoad() {
        super.viewDidLoad()
        
        //Image Picker
        imagePickerController.delegate = self
    
        //Image TapGesture
        let imageTapGesture = UITapGestureRecognizer(target: self, action: #selector(FirstViewController.imageTapped))
        imageTapGesture.numberOfTapsRequired = 1
        placeImage.addGestureRecognizer(imageTapGesture)
        placeImage.isUserInteractionEnabled = true
        
        //Core location
        locationManager.delegate = self
        locationManager.requestWhenInUseAuthorization()
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.startUpdatingLocation()
        
        
    }
```

We will let the ImagePicker run when we tap the image view. First, I will explain how to locate the user and get the address.

```swift
import Foundation
import CoreLocation


extension CLGeocoder {
    
    func getAddress(location: CLLocation, completion: @escaping (String) -> ()){
        
        var addressString : String = ""
        
        self.reverseGeocodeLocation(location) { (placemarks, error) in
            
            if (error != nil) {
                print("getAddress Error :\(String(describing: error?.localizedDescription))")
                completion(addressString)
            }
            
            let placemark = placemarks! as [CLPlacemark]
            if placemark.count > 0 {
                
                let currentAddress = placemarks![0]
                
                if currentAddress.subThoroughfare != nil {
                    addressString = addressString + currentAddress.subThoroughfare! + ", "
                }
                if currentAddress.thoroughfare != nil {
                    addressString = addressString + currentAddress.thoroughfare! + ", "
                }
                if currentAddress.subLocality != nil {
                    addressString = addressString + currentAddress.subLocality! + ", "
                }
                if currentAddress.locality != nil {
                    addressString = addressString + currentAddress.locality! + ", "
                }
                if currentAddress.postalCode != nil {
                    addressString = addressString + currentAddress.postalCode! + ", "
                }
                if currentAddress.country != nil {
                    addressString = addressString + currentAddress.country! + ""
                }
                
                completion(addressString)
            }
        }
    }
}
```

I created an extension that uses gps coordinates and a geocoder to get the address of the current location as a String.



```swift
extension FirstViewController : CLLocationManagerDelegate {
    
    func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {

        //1. When the location information is accepted, it starts to obtain information from the GPS.
        if status == .authorizedWhenInUse {
            
            locationManager.startUpdatingLocation()
   
        }
    }
    
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        
        let coordinate = locations as NSArray
        let coordinate2D = coordinate.lastObject as! CLLocation
        
        //Upate currentLocation
        currentLocation = coordinate2D
      
        
        //2. Once the location information is obtained, it stops receiving information from the GPS.
        if locations.first != nil {
            locationManager.stopUpdatingLocation()
        }

        //3. We get address information about the current location through GeoCoder Extension which we made before.
        geocoder.getAddress(location: coordinate2D) { (address) in
            self.addressLabel.text = address
        }
        
      
    }
    
}
```

Receive CLLocation events through CLLocationManagerDelegate. Depending on whether you have access to location information, the user's location information is received from the GPS, and the address is displayed on the UILabel.



```swift
extension FirstViewController : UIImagePickerControllerDelegate, UINavigationControllerDelegate {
    
    //Request access to a photo album
    func authorizeToAlbum(completion:@escaping (Bool)->Void) {
        
        if PHPhotoLibrary.authorizationStatus() != .authorized {
            PHPhotoLibrary.requestAuthorization({ (status) in
                if status == .authorized {
                    DispatchQueue.main.async(execute: {
                        completion(true)
                    })
                } else {
                    DispatchQueue.main.async(execute: {
                        completion(false)
                    })
                }
            })
            
        } else {
            DispatchQueue.main.async(execute: {
                completion(true)
            })
        }
    }
    
    //When you tap on the image, it presents an ImagePickerController.
    @objc func imageTapped(recognizer:UITapGestureRecognizer){
        
        self.authorizeToAlbum { (authorized) in
            if authorized == true {
                
                self.imagePickerController.sourceType = .photoLibrary
                self.imagePickerController.allowsEditing = true
                self.present(self.imagePickerController, animated: true, completion: nil)
            
            }
        }
        
        
    }
    
    //When the user completes the selection of the image, the UIImagePickerController disappears and is called at that time.
    @objc func imagePickerController(_ picker: UIImagePickerController, didFinishPickingImage image: UIImage!, editingInfo: [NSObject : AnyObject]!) {
        placeImage.image = image
        self.dismiss(animated: true, completion: nil)
    }

    //This function is called when Cancel is selected without image selection
    @objc func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
        picker.dismiss(animated: true, completion: nil)
    }
    
}
```

Before presenting ImagePickerController, make sure that you have access to the user album. Then it will present the ImagePickerController.



```swift
@IBAction func savePlaceTapped(_ sender: Any) {
    
        //1. Check if information about place is entered without missing
        if (addressLabel.text?.isEmpty)! || (nameField.text?.isEmpty)! || (placeImage.image == nil) {
    
            showAlertMessage(title: "Wait..!", message: "Please check the name and photo of the place.", completion: nil)
            
            return
        }
        
        //2. Converts UIImage to JPG data and unwrap optional CLLocation data.
        guard let image = UIImageJPEGRepresentation(placeImage.image!, 1), let location = currentLocation else { return }

        
        //3. Create a MyPlace object to be stored on the parse server
        let currentPlace =  MyPlace()
        
        //4. Save the entered data to the MyPlace object
        currentPlace.address = addressLabel.text! as NSString
        currentPlace.img = PFFile(data:image)
        currentPlace.location = PFGeoPoint(location: location)
        currentPlace.owner = User.current()
        currentPlace.name = nameField.text! as NSString

        
        let spiningActivity = MBProgressHUD.showAdded(to: self.view, animated: true)
        spiningActivity.label.text = "Please wait"
        spiningActivity.detailsLabel.text = "Uploading in to the server"
        
        
        //5. Save the MyPlace object to Parse Server. It works as a background thread, and when it is done, it confirms that it is saved through the completion handler.
        currentPlace.saveInBackground { (success, error) in
            
            MBProgressHUD.hide(for: self.view, animated: true)
            
            if error != nil {
                
                self.showAlertMessage(title: "Upload failed", message: (error?.localizedDescription)!, completion: nil)
                
            }else {
                
                self.showAlertMessage(title: "Upload success", message: "Place information has been successfully stored on the server.", completion: { (action) in
                
                //6. Clear fileds
                self.clearFields()
                
                })
            
            }
        }
        
    }
    
    func clearFields(){
        
        placeImage.image = #imageLiteral(resourceName: "placeholder")
        nameField.text = nil
        
    }
```

Congratulations! You are now ready to store your data on Parse Server.

![Upload](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/upload.gif)

Let's build and run, and We can check that the information about the place is stored in Parse Server successfully.



![myplace saved](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/myplace.png)

Try connecting to Parse Server and see if the data we entered is stored in the database. We confirmed that it was properly stored as above. Now let's create MapView, the last step of this tutorial!

## The Last Step, Let's Create MapView

First we will go to Storyboard and set Autolayout for MapView. Our app has `FirstViewController` and `SecondViewController` associated with the `Tab controller`. `FirstViewController` was used to store the `MyPlace` described in the previous step. `SecondViewController` will be used for MapView.

![SecondVC](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/secondVC.png)

First, let's choose `SecondViewController`.



![Change to Pulley](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/secondVC change to Pulley.png)

Change the class of `SecondViewController` to `PulleyViewController` as shown in the upper right.



![ContainerView](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/containerView.png)

Add two `Container Views` to `SecondViewController`.

![ContainerView](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/selectContainerView.png)

Name the two `Container Views` as `Primary Container` and `Drawer Conatainer`. `Primary Container` will be used as a `MapView` and `Drawer Container` will be used as a `Table View`.

I will now explain how to set up Autolayout with an screenshot. Take a look at the following screenshots below and follow the Autolayout settings.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/step1.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/step2.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/step3.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/step4.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/step5.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/step6.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/step7.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/step8.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/step9.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/step10.png)

You have now finished the basic Autolayout settings. 



![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/createVC.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/mapVC.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/searchVC.png)

Create a UIViewController associated with the Primary Container and the Drawer Container respectively. We just created two new UIViewControllers.

- `MapViewController` : This is the ViewController associated with the Primary Container.
- `SearchViewController` : This is the ViewController associated with the Drawer Container.
- ​

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/overviewStoryboard_step1.png)

The overall configuration of `ViewController` is as above. Before applying a detailed UI, We will simply check that the `Pulley ViewController` is working properly.



![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/primaryContainer.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/primaryContainer_outlet.png)

To make `PulleyViewController` work properly, connect the `Referencing Outlet` like above.



![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/drawerContainer.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/drawerContainer_outlet.png)

`DrawerContainer` also connects `Referencing Outlet` like `Primary Container`. The basic preparation for using `Pulley ViewController` is now complete.



![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/map.gif)

This is what we have done so far. As you can see above, the `Primary Container` displays a `MapView` and the `Drawer Container` is a mint color area.

Let's move on to `Primay Container` a bit more. Here are the features that will be added to the `MapView`:

- Add a button back to your current location
- Show saved places on map



![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/mapCapabilities.png)

In order to use the function to display the current position on the map, it is necessary to set it as shown in the figure above.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/buttonPosition.png)

Sets the location of the button that performs the function to update the current position in MapViewController.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/reverseItem.png)

Select Bottom Constraint for Button and select **Reverse First and Second Item**

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/bottomConstraint.png)

Create an Outlet for the Bottom Constraint. This is an important variable to be used to prevent the `Location Button` from being hidden when the `Drawer` moves up and down.

```swift
import UIKit
import Pulley
import MapKit
import CoreLocation

class MapViewController: UIViewController {
    
    //buttonBottom and locationBottomConstraint declared to maintain the spacing of the Location Button
    fileprivate let buttonBottom: CGFloat = 16.0
    @IBOutlet weak var locationBottomConstraint: NSLayoutConstraint!
    
    @IBOutlet weak var mapView: MKMapView!
    
    var currentLocation : CLLocation?
    let locationManager = CLLocationManager()
    
    // Updates the center view of the map based on your current location.
    @IBAction func setCurrentLocation(_ sender: Any) {
        
        guard let location = currentLocation else { return }
        
        let span = MKCoordinateSpanMake(0.005, 0.005)
        let region = MKCoordinateRegion(center: location.coordinate, span: span)
        mapView.setRegion(region, animated: true)
        
    }
    
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        locationManager.delegate = self
        locationManager.requestWhenInUseAuthorization()
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.startUpdatingLocation()
        
    }
    
}
```

The `bottomButton` is used to maintain the bottom space of the Location Button relative to the top of the Drawer View.

`setCurrentLocation` is a function that zooms the map based on the current device location. The parameter value 1 of the  [MKCoordinateSpanMake](https://developer.apple.com/documentation/mapkit/1452263-mkcoordinatespanmake)  function means 111 km (64 miles). As shown in the code above, the zoom level is set to 0.005, which is 0.5km or 500m.

```swift
extension MapViewController: PulleyPrimaryContentControllerDelegate {
    
    func drawerChangedDistanceFromBottom(drawer: PulleyViewController, distance: CGFloat, bottomSafeArea: CGFloat)
    {

        guard drawer.currentDisplayMode == .bottomDrawer else {

            locationBottomConstraint.constant = bottomSafeArea
            return
        }
        
        // When the drawer is up to the middle height of the ViewController
        if distance <= 268.0 + bottomSafeArea
        {
            locationBottomConstraint.constant = distance - bottomSafeArea + buttonBottom
        }
        ////When the drawer comes up to the top
        else
        {
            locationBottomConstraint.constant = 268.0 + buttonBottom
        }
    }
}
```

 `drawerChangedDistanceFromBottom` is used to update the location of the UI, such as the `Location Button`, which can be affected when the height of the drawer changes.

```swift
extension MapViewController: CLLocationManagerDelegate {
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        
        let coordinate = locations as NSArray
        let coordinate2D = coordinate.lastObject as! CLLocation
        
        currentLocation = coordinate2D
        
        //zooming center
        if let location = locations.first {
            
            let span = MKCoordinateSpanMake(0.005, 0.005)
            let region = MKCoordinateRegion(center: location.coordinate, span: span)
            mapView.setRegion(region, animated: true)
        }
        
        
    }
    
    func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
        if status == .authorizedWhenInUse {
            locationManager.requestLocation()
        }
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        print("error:: (error)")
    }
    
}
```

Almost the same as the code described earlier in `FirstViewController`. One difference is that when the Location information is updated, the center of the map is updated based on the current location.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/map2.gif)

This is the result of running the code described above. First, when the map is loaded, the center of the map is updated based on the current location. When the drawer moves up and down, the `Location Button` is also moved so that it keeps a constant distance based on the top of the drawer. Finally, when you tap the `Location Button`, the center of the map is updated.

`MapViewController` is almost implemented. Now let's implement a `Map` that displays the **location information** we have stored on the parse server.

## Draw a path on a map

Let's save three places in the city of London and mark the route on the map.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/latlon.png) 

You can see latitude and longitude as above by selecting three places in London city from [latlong.net](https://www.latlong.net/) 

Let's save the place three times in FirstViewController. However, our app will automatically save the latitude and longitude based on the current location, so we'll modify it in Parse-Server for demo purposes.



![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/mapview/updatelatlon.png)

Connect to Parse Server and modify the Location value of MyPlace to the value confirmed by [latlong.net](https://www.latlong.net/)

Now let's create the `LoadPlaceData` protocol, which takes place information from Parse Server.

```swift
import Foundation
import UIKit
import Parse
import MapKit


protocol LoadPlaceData : class {
    
    var places: [MyPlace]{ get set }
    var placeAnnotations : [MyPlaceAnnotation] { get set }
    var routeLocations: [[CLLocation:CLLocation]] { get set }
    var routes : [MKRoute] { get set }
    
    func loadData(completion: @escaping (Bool) -> ())
    func showAnnotations(completion: @escaping (Bool) -> ())
    func calculateRoute(completion: @escaping (Bool) -> ())
}
```

The four properties are described below

`places` : Fetch and save places saved in Parse Server

`placeAnnotations`: Used to mark places on the map

`routeLocations`: The starting point and the destination information are required to draw the route on the map. Our app stores multiple places and displays the full path on the map. Therefore, several starting points and destination information are needed.

`routes` : It is used to store the result of calculating route for each `routeLocation`

After describing `MyPlaceAnnotation`, I will explain the function for `loadData`, `showAnnotation` and `calculateRoute`.

```swift
import Foundation
import MapKit

class MyPlaceAnnotation : NSObject, MKAnnotation {

    let name: String
    let address: String
    var coordinate: CLLocationCoordinate2D
    
    
    init(name: String, address: String, coordinate: CLLocationCoordinate2D) {
        self.name = name
        self.address = address
        self.coordinate = coordinate
        
        super.init()
    }
    
    
}
```

We need an MKAnnotation object to mark a place on the map. We first request the place data from the Parse Server and store the result in the MyPlace object. We will then change the MyPlace object to a MyPlaceAnnotation object and mark the place on the map.

```swift
func loadData(completion: @escaping (Bool) -> ()) {
    
        guard let currentUser = User.current() else { return }
        
    	//1. Request location information from Parse Server
        let query = PFQuery(className: "MyPlace")
        query.whereKey("owner", equalTo:currentUser)
        query.findObjectsInBackground { (results, error) in
        
            //Error
            if error != nil {
                completion(false)
                
            // Success
            }else {
                
                guard let fetchedResults = results as? [MyPlace] else { return }
                //2. Store place information from Parse Server
                for result in fetchedResults {
                    self.places.append(result)
                }
                
                completion(true)
                
            }
            
        }
        
    }
```

`PFQuery` is used to request data stored in Parse Server. It receives the data generated by the user logged in via `whereKey`.

`loadData` is called inside the `showAnnotaions` function.

```swift
func showAnnotations(completion: @escaping (Bool) -> ()){
        
        loadData{ [unowned self] (success) in
            
            if success {
                for place in self.places {
                    
                    guard let placeName = place.name as String?, let placeAddress = place.address as String?, let placeCoordinate = place.location else { return }
                    
                    let annotation = MyPlaceAnnotation(name: placeName, address: placeAddress, coordinate: CLLocationCoordinate2D(latitude: placeCoordinate.latitude, longitude: placeCoordinate.longitude))
                    
                    self.placeAnnotations.append(annotation)
   
                }
                
                completion(true)
                
            }
            
        }
        
    }
```

After successfully saving the place to the `MyPlace` object via loadData, save it back as a `MyPlaceAnnotation` object.

Now we are ready to mark the place on the map. Finally, let's look at how to calculate the path.

| Source | Destination |
| ------ | ----------- |
| Market | Opera       |
| Opera  | Bridge      |

I saved three places as described above. We will look at how to draw all the paths to these three places on the map.



```swift
func calculateRoute(completion: @escaping (Bool) -> ()){
        
        //1. loop in 0...2
        for i in 0...placeAnnotations.count - 1 {
        
            guard i+1 <= placeAnnotations.count - 1 else { break }
            
            let source = CLLocation(latitude: placeAnnotations[i].coordinate.latitude, longitude: placeAnnotations[i].coordinate.longitude)
            let destination = CLLocation(latitude: placeAnnotations[i+1].coordinate.latitude, longitude: placeAnnotations[i+1].coordinate.longitude)
            
            //2. Save source and destination
            routeLocations.append([source : destination])
            
            //3. Preparations for calculating paths using MKDirectionsRequest
            let directionRequest = MKDirectionsRequest()
            let startPoint = MKPlacemark(coordinate: placeAnnotations[i].coordinate)
            let endPoint = MKPlacemark(coordinate: placeAnnotations[i+1].coordinate)
            
            directionRequest.source = MKMapItem(placemark: startPoint)
            directionRequest.destination = MKMapItem(placemark: endPoint)
            directionRequest.transportType = .automobile
            
            
            let directions = MKDirections(request: directionRequest)
            
            //4. Calculate path
            directions.calculate { (routeResponse, routeError) -> Void in
                
                guard let routeResponse = routeResponse else {
                    if let routeError = routeError {
                        print("Error: \(routeError)")
                    }
                    
                    return
                }
                
                self.routes.append(routeResponse.routes[0])
                
                //5. Once all paths have been calculated, call the completion handler
                if i+1 == self.placeAnnotations.count - 1 {
                    
                    completion(true)
                }   
            }   
        } 
    }
```

It seems a bit complicated, but I commented on the important parts of each step in the code.

In the second and third annotation, the source and destination information are recorded in the routeLocations as shown in the table above, and the route is calculated for each route.



```swift
import Foundation
import UIKit
import Parse
import MapKit


protocol LoadPlaceData : class {
    
    var places: [MyPlace]{ get set }
    var placeAnnotations : [MyPlaceAnnotation] { get set }
    var routeLocations: [[CLLocation:CLLocation]] { get set }
    var routes : [MKRoute] { get set }
    
    func loadData(completion: @escaping (Bool) -> ())
    func showAnnotations(completion: @escaping (Bool) -> ())
    func calculateRoute(completion: @escaping (Bool) -> ())
}


extension LoadPlaceData where Self: UIViewController {
    
    
    func loadData(completion: @escaping (Bool) -> ()) {
    
        guard let currentUser = User.current() else { return }
        
        let query = PFQuery(className: "MyPlace")
        query.whereKey("owner", equalTo:currentUser)
        query.findObjectsInBackground { (results, error) in
        
            //Error
            if error != nil {
                completion(false)
                
            // Success
            }else {
                
                guard let fetchedResults = results as? [MyPlace] else { return }
                for result in fetchedResults {
                    self.places.append(result)
                }
                
                completion(true)
                
            }
            
        }
        
    }
    
    func showAnnotations(completion: @escaping (Bool) -> ()){
        
        loadData{ [unowned self] (success) in
            
            if success {
                for place in self.places {
                    
                    guard let placeName = place.name as String?, let placeAddress = place.address as String?, let placeCoordinate = place.location else { return }
                    
                    let annotation = MyPlaceAnnotation(name: placeName, address: placeAddress, coordinate: CLLocationCoordinate2D(latitude: placeCoordinate.latitude, longitude: placeCoordinate.longitude))
                    
                    self.placeAnnotations.append(annotation)
   
                }
                
                completion(true)
                
            }
            
        }
        
    }
    
    func calculateRoute(completion: @escaping (Bool) -> ()){
        
        
        for i in 0...placeAnnotations.count - 1 {
        
            guard i+1 <= placeAnnotations.count - 1 else { break }
            
            let source = CLLocation(latitude: placeAnnotations[i].coordinate.latitude, longitude: placeAnnotations[i].coordinate.longitude)
            let destination = CLLocation(latitude: placeAnnotations[i+1].coordinate.latitude, longitude: placeAnnotations[i+1].coordinate.longitude)
            
            routeLocations.append([source : destination])
            
            
            
            let directionRequest = MKDirectionsRequest()
            let startPoint = MKPlacemark(coordinate: placeAnnotations[i].coordinate)
            let endPoint = MKPlacemark(coordinate: placeAnnotations[i+1].coordinate)
            
            directionRequest.source = MKMapItem(placemark: startPoint)
            directionRequest.destination = MKMapItem(placemark: endPoint)
            directionRequest.transportType = .automobile
            
            
            let directions = MKDirections(request: directionRequest)
            
            directions.calculate { (routeResponse, routeError) -> Void in
                
                guard let routeResponse = routeResponse else {
                    if let routeError = routeError {
                        print("Error: \(routeError)")
                    }
                    
                    return
                }
                
                self.routes.append(routeResponse.routes[0])
                
                if i+1 == self.placeAnnotations.count - 1 {
                    
                    completion(true)
                }
                
            }
        }
        
    }

}

```

The above code is all about `LoadPlaceData.swift`

Now go back to `MapViewController` and use the `LoadPlaceData` protocol described above.

```swift
class MapViewController: UIViewController, LoadPlaceData {
    
    
    var places: [MyPlace] = []
    var placeAnnotations: [MyPlaceAnnotation] = []
    var routeLocations: [[CLLocation : CLLocation]] = []
    var routes: [MKRoute] = []
```

Let's use the `LoadPlaceData` protocol in `MapViewController`.  First, declare the properties of the `LoadPlaceData` protocol as above and initialize it.

```swift
override func viewDidLoad() {
        super.viewDidLoad()
        
        locationManager.delegate = self
        locationManager.requestWhenInUseAuthorization()
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.startUpdatingLocation()
        

        //set annotations
        showAnnotations { [unowned self](success) in
            if success {
                self.mapView.addAnnotations(self.placeAnnotations)
                
                self.calculateRoute { [unowned self](success) in
                    
                    if success {
                        
                        for route in self.routes {
                            //draw routes
                            self.mapView.add(route.polyline, level: MKOverlayLevel.aboveRoads)
                        }
                        
                    }
                    
                }
            }
        }
    }
```

In `viewDidLoad` as above, call the `LoadPlaceData` protocol function to display the path and place on the map.

```swift
extension MapViewController: MKMapViewDelegate {
    
    func mapView(_ mapView: MKMapView, rendererFor overlay: MKOverlay) -> MKOverlayRenderer {
        
        let renderer = MKPolylineRenderer(overlay: overlay)
        renderer.strokeColor = UIColor.blue
        renderer.lineWidth = 3.0
        
        return renderer
    }
    
}
```

To display the path on the map, you need to call `mapView (_ mapView: MKMapView, rendererFor overlay: MKOverlay)` as above.

The `MapViewController` implementation is now complete. Before you implement `SearchViewController`, check out what you have developed so far.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/map3.gif)

Amazing! All three places we have saved on the map are displayed correctly. Not only that, but all the paths are well marked. 

Now, finally, let's implement a part of searching around a place like Apple Map.

## Search for nearby places using MKLocalSearch

In this section, we will configure the basic UI using SearchBar and TableView. And we will use MKLocalSearch to search for nearby restaurants. First, let's set the UI of SearchViewController.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/searchView1.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/searchView2.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/searchView3.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/searchView4.png)

The above steps are the process of adding a UIView. Next we will add `SearchBar`



![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/searchbar1.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/searchbar2.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/handlebar1.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/handlebar2.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/handlebar3.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/handlebar4.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/handlebar5.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/handlebar6.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/handlebar 7.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/handlebar 8.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/handlebar 9.png)

The above steps show how to arrange `SearchBar` and `Handle` images using Autolayout.

Next, place the `TableView` at the bottom of the `SearchBar`

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/table1.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/table2.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/table3.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/table4.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/table5.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/table6.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/table7.png)

We have finished placing the `TableVeiw`. Now we will drag and drop the `TableViewCell` onto the `TableView` and place a `UILabel` to display the `name`, `address`, and `distance` of the place.



![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/tablecell1.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/tablecell2.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/stackview.png)

Place two `UILabels` in the `UITableViewCell` and click `Embeded in StackView`

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/tablecell3.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/tablecell4.png)

Set the `UILabel`'s Font to display the name of the place as **Bold**

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/tablecell5.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/stackview.png)

Select the `StackView` created above and the newly added `UILabel` and click `Embeded in StackView`.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/tablecell6.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/tablecell7.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/tablecell8.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/address1.png)

Select the `UILabel` to display the Address and set the Lines to 2 as shown in the upper right corner. If the address is long, it can be displayed in 2 lines.



![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/tablecell9.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/tablecell10.png)

I have almost completed the `UILabel` configuration so far. Now let's add a `UIView` to seperate the cell.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/border1.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/border2.png)

Now let's create a class for `UITableViewCell`.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/class1.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/class2.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/class3.png)

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/class4.png)

Finally, We will set up the Delegate of `TableView` and `SearchBar` and write the code.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/tableviewDelegate.png)

Set the DataSource and Delegate as above.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/searchDelegate.png)

We also sets the delegate for SearchBar as shown above.

Finally, all settings are finished. Now we are going to write the code.

```swift
import UIKit

class PlaceCell: UITableViewCell {

    @IBOutlet weak var nameLabel: UILabel!
    @IBOutlet weak var addressLabel: UILabel!
    @IBOutlet weak var distanceLabel: UILabel!
    
    
    override func awakeFromNib() {
        super.awakeFromNib()
        // Initialization code
    }
}
```

It is the code of `PlaceCell` which is the Custom Class of `UITableViewCell`. We connected the `IBOutlet` to the` UILabels` we created earlier in the `storyboard` This cell will be used to display the place search results in a `TableView`.

```swift
import UIKit
import Pulley
import CoreLocation
import MapKit
```

Import the library and framework to be used in `SearchViewController` as above.

```swift
  @IBOutlet weak var searchBar: UISearchBar!
    @IBOutlet weak var tableView: UITableView!
    
    //Used to obtain the coordinate value of a place that matches address
    let geocoder = CLGeocoder()
   
	//Set true if you entered a keyword in SearchBar; otherwise, set false.
    var isSearchOn = Bool()

	//Used to search nearby places
    var searchCompleter = MKLocalSearchCompleter()
    var searchResults = [MKLocalSearchCompletion]()
    
    //Used to measure the distance between the current location and the searched location
    let df = MKDistanceFormatter()

    var currentLocation : CLLocation!
    let locationManager = CLLocationManager()
    
    //Save nearby restaurants
    var placeItems:[MKMapItem?] = []
    
    //Save search results by keyword
    var searchPlaceResults = [NearbyPlace]()
```

These are variables used in `SearchViewController`. The meaning of each variable is explained in the annotation.

```swift
override func viewDidLoad() {
        super.viewDidLoad()

        //set SearchBar Style
        searchBar.barTintColor = UIColor.white
        searchBar.backgroundColor = UIColor.white
        searchBar.backgroundImage = UIImage()
        searchBar.isTranslucent = true
        
        if let textField = searchBar.value(forKey: "_searchField") as? UITextField {
            textField.backgroundColor = UIColor(red: 248/255, green: 248/255, blue: 248/255, alpha: 1.0)
        }
        
        isSearchOn = false
        searchCompleter.delegate = self
        
        //core location
        locationManager.delegate = self
        locationManager.requestWhenInUseAuthorization()
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.startUpdatingLocation()
        
        //e.g : km instead of kilometer
        df.unitStyle = .abbreviated
        
        setupTableView()
    }
```

When the `SearchViewController` is loaded, it changes the style of the `SearchBar` and sets the default values for the variables declared above. Finally, we call a function that sets the `TableView` to display the retrieved result.

```swift
func setupTableView(){
        
        self.tableView.estimatedRowHeight = 70
        self.tableView.rowHeight = UITableViewAutomaticDimension
        self.tableView.separatorStyle = .none
        
    }
```

Since the cell of the `TableView` may vary depending on the length of the address, we set `rowHeight` to` UITableViewAutomaticDimension`. And since we used `UIView` as the **seperator**, we do not use the `TableView` **seperator**.



```swift
extension SearchViewController: UISearchBarDelegate {
    
    //1. When you tap searchBar, SearchViewController goes up from the bottom to the top.
    func searchBarTextDidBeginEditing(_ searchBar: UISearchBar) {
        
        if let drawerVC = self.parent as? PulleyViewController
        {
            drawerVC.setDrawerPosition(position: .open, animated: true)
        }
    }
    
    //2. This function is called when you enter text. If there is an input character, it searches for a nearby place. To avoid being called each time a character is entered, it starts searching for nearby places when the input is paused.
    func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String){
        
        NSObject.cancelPreviousPerformRequests(withTarget: self, selector: #selector(locationTextDidChange), object: nil)
        self.perform(#selector(locationTextDidChange), with: nil, afterDelay: 0.5)
        
    }
    
    @objc func locationTextDidChange(){
        
        if !(searchBar.text?.isEmpty)! {
            
            isSearchOn = true
            searchCompleter.filterType = .locationsOnly
            searchCompleter.queryFragment = searchBar.text!
            
        }else {
            
            isSearchOn = false
            searchPlaceResults.removeAll()
            tableView.reloadData()
        }
        
    }
    
    func searchBarCancelButtonClicked(_ searchBar: UISearchBar) {
        
        isSearchOn = false
        searchPlaceResults.removeAll()
        tableView.reloadData()
        
    }
}
```

This is the code associated with `SearchBar`. The search is performed only when there is a character input by the user. If nothing is entered in the `SearchBar` or if you delete all the entered character, nothing is displayed in the `TableView`.

```swift
func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        
        let coordinate = locations as NSArray
        let coordinate2D = coordinate.lastObject as! CLLocation
        
        //upate currentLocation
        currentLocation = coordinate2D
        
        if locations.first != nil {
            locationManager.stopUpdatingLocation()
        }
        
        //It is a variable to store if there is a restaurant around the current location. It will be called every time the location information of the user is updated, so it is initialized before performing the search.
        self.placeItems.removeAll()
        
        
        //We set up to search restaurants near 200M around the current location.
        let request = MKLocalSearchRequest()
        request.region = MKCoordinateRegionMakeWithDistance(coordinate2D.coordinate, 200, 200)
        request.naturalLanguageQuery = "restaurant"
        searchCompleter.region = MKCoordinateRegionMakeWithDistance(currentLocation.coordinate, 200, 200)
        
        
        //Start searching for restaurants nearby. If the restaurant is found, store it in placeItems.
        let search = MKLocalSearch(request: request)
        search.start { response, _ in
            guard let response = response else {
                return
            }
            self.placeItems = response.mapItems
            self.tableView.reloadData()
        }   
    }
```

This function is performed when the current location location is successfully received from the GPS. Find a restaurant within a 200m radius around your current location and display it in the `TableView`.

```swift
func completerDidUpdateResults(_ completer: MKLocalSearchCompleter) {
        
    	//Save retrieved results
        searchResults = completer.results
        self.searchPlaceResults.removeAll()
    
        for (index, place) in searchResults.enumerated() {
    
            let searchPlaceResult = NearbyPlace(mkCompletion: place, mapItem: nil, distanceRaw: nil, distance: nil, coordinate: nil)
            self.searchPlaceResults.append(searchPlaceResult)
            
            ////Check the GPS coordinate using the address information obtained from the search result. Then calculate and store the distance between the current location and the location.
            self.getCoordinate(addressString: place.title, completionHandler: { (location, error) in
            
                if error == nil {
                    //Calculate distance
                    let distanceRaw = self.currentLocation.distance(from: CLLocation(latitude: location.latitude, longitude: location.longitude))
                    //Change the calculated distance value to a format that is easy for the user to read. For example, km or mi.
                    let distance = self.df.string(fromDistance: distanceRaw)
                    
                    self.searchPlaceResults[index].distanceRaw = distanceRaw
                    self.searchPlaceResults[index].distance = distance
                    self.searchPlaceResults[index].fetched = true

                }
            })
        }
        
    	
        DispatchQueue.main.async {
            self.tableView.reloadData()
        }
    }
```

The above function is called when there is a result when searching by a place name.

```swift
func getCoordinate( addressString : String, completionHandler: @escaping(CLLocationCoordinate2D, Error?) -> Void ){
        
        let geocoder = CLGeocoder()
        geocoder.geocodeAddressString(addressString) { (placemarks, error) in
            if error == nil {
                if let placemark = placemarks?[0] {
                    let location = placemark.location!
                    completionHandler(location.coordinate, nil)
                    return
                }
            }
            completionHandler(kCLLocationCoordinate2DInvalid, error as Error?)
        }
    }
```

The above functions are used to obtain coordinate for a place using a geocoder.

Finally, let's look at the code that displays the searched places in the `TableViewCell`.

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        
        let cell = tableView.dequeueReusableCell(withIdentifier: "PlaceCell", for: indexPath) as! PlaceCell
        
        var name = String()
        var address = String()
        var distance = String()
        var distanceRaw = Double()
        
        if isSearchOn {
            
            let fetchSearchItem = searchPlaceResults[indexPath.row]
            
            name = fetchSearchItem.mkCompletion.title
            address = fetchSearchItem.mkCompletion.subtitle
            
            if let locationDistance = fetchSearchItem.distance {
                distance = locationDistance
            }
            
        }else {
            
            if let fetchPlaceItem = placeItems[indexPath.row] {
                
                name = fetchPlaceItem.name ?? ""
                address = fetchPlaceItem.placemark.title ?? ""
                
                //get meters
                distanceRaw = currentLocation.distance(from: CLLocation(latitude: fetchPlaceItem.placemark.coordinate.latitude, longitude: fetchPlaceItem.placemark.coordinate.longitude))
                
                distance = df.string(fromDistance: distanceRaw)
                
            }
            
        }
        
        cell.nameLabel.text = name
        cell.addressLabel.text = address
        cell.distanceLabel.text = distance
        
        return cell
    }
```

In the code above, we use `searchPlaceResults` when searching for a place through SearchBar. If not, use `placeItems`.

`placeItems`: When the location information of the user is updated, it is a variable that stores the restaurants within the radius of 200 by default based on the current location.

`searchPlaceResults`: The variable that stores the result when the user enters the `Searchbar` to search for a specific place

Wow finally made it all. Let's run our app.

![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/map4.gif)

As you can see, The functionality we want is implemented correctly. It was a really long tutorial, but thanks for reading so far.

But there seems to be one more improvement. There is a problem that the keyboard covers part of the search results. Let's solve it simple.

```swift
override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        locationManager.startUpdatingLocation()
        
        NotificationCenter.default.addObserver(self, selector: #selector(SearchViewController.keyboardWillShow(_:)), name: NSNotification.Name.UIKeyboardWillShow, object: nil)

        
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        
        NotificationCenter.default.removeObserver(self, name: .UIKeyboardWillShow , object: nil)
        
    }
```

```swift
@objc func keyboardWillShow(_ notification : Notification){
        
        guard let userInfo = notification.userInfo else { return }
        let keyboard = (userInfo[UIKeyboardFrameEndUserInfoKey] as! NSValue).cgRectValue
        
        var contentInsets = UIEdgeInsets()
        
        if UIInterfaceOrientation.portrait.isPortrait {
            
            contentInsets = UIEdgeInsetsMake(0.0, 0.0, (keyboard.height + 20), 0.0)
            
        }else {
            
            contentInsets = UIEdgeInsetsMake(0.0, 0.0, (keyboard.width), 0.0)
            
        }
        
        self.tableView.contentInset = contentInsets
        self.tableView.scrollIndicatorInsets = contentInsets
        
        
    }
```

Let's fix it by adding an observer for the `keyboardWillShow`.
When the keyboard appears, set the content inset of the tableview to the height of the keyboard.

Now all implementations are complete. Let's run the app again and check it out.



![](/Users/GurumCEO/Dropbox (Personal)/Typora/Apple Map/search/mapFinal.gif)

That's all. I put the project code on github.

