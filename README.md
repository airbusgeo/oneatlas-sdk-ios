# OneAtlas iOS SDK

## Prerequisites

You need a OneAtlas developer account. To create it, go to our [Developer Site](https://oneatlas.airbus.com).

Your development machine should have the latest version of [XCode](https://developer.apple.com/xcode/) installed.

The recommended way of installing our SDK is through [CocoaPods](https://cocoapods.org), a widely used dependency management system for iOS development.

To install the CocoaPods CLI on your development machine, run in the Terminal:

```
$ sudo gem install cocoapods
```


## Add the SDK to your project (through CocoaPods)

Navigate to your Xcode project's folder and initialize your ```Podfile``` sing the following command:

```
$ pod init
```

Add the following code to this file (don't forget to replace ```your_target``` with your Xcode target's name):

```ruby
target 'your_target' do
  pod 'OneAtlas'

  # add any other pods below this line
end

```

By running this command in the Terminal, CocoaPods will create the matching Xcode workspace:

```shell
$ pod install
```

You can now open the generated Xcode workspace:

```shell
$ open <your_project>.xcworkspace
```

<!--
## Add the SDK to your project (through Carthage)
To install Carthage on your development machine, run in the Terminal:
```shell
$ brew install carthage
```
Navigate to your project's folder and create an empty file named ```Cartfile```.
Add the following code to this file:

```shell
binary "https://ios.tls-pangea.gq/OneAtlas.json"
```

Run this in the Terminal, Carthage will download our SDK into the ```Carthage/Build/iOS``` folder.

```shell
$ carthage update
```

Go to Xcode, select the ```General``` tab of your project's properties, scroll down to the ```Embedded Binaries``` section, then drag the ```OneAtlas.framework``` file from the Finder to ```Embedded Binaries```.
In the dialog box that will appear, click on ```Finish``` (leave all options to default).
-->

## Use OneAtlas SDK into your project

Once you've installed the framework, just import the main header file:

```objective_c
// Objective C
#import <OneAtlas/OneAtlas.h>
```

```swift
// Swift
import OneAtlas
```

## Quick Start
```objective_c
// authenticate with your API Key on AAA realm
[[[OneAtlas sharedInstance] authenticateService] loginWithAPIKey:@"<your_api_key>" clientID:EAAAClientID block:^(OAError * aError) {
    // check for errors and continue
    if(aError) {
        // handle error
    }
    else {
        // retrieve the ID of Living Library Workspace
        [[[OneAtlas sharedInstance] dataService] getMeWithBlock:^(OAUser *aUser, OAError *aError) {
            if(aError) {
                // handle error
            }
            else {
                // create a filter based on a location
                OAPoint * location = [OAPoint pointFromLatitude:43.0 longitude:3.0];
                OASearchFilter * filter = [OASearchFilter new];
                [filter setGeometry:location];
                
                // create a sort object
                OASort * sort = [OASort new];
                [sort addSortingCriteria:ESortAcquisitionDate order:ESortOrderDescending];

                // perform a search using those criterias
                OAPagination * page = [OAPagination makePage:0 itemsPerPage:10];
                [[[OneAtlas sharedInstance] searchService] openSearchWithPagination:page workspaceID:aUser.contract.workspaceID filter:filter sort:sort block:^(NSArray<OAFeature *> *aFeatures, OAPagination *aPagination, OAError *aError) {
                    if(aError) {
                        // handle error
                    }
                    else {
                        // handle your search results
                    }
                }];
            }
        }];
    }
}];
```

```swift
// authenticate with your API Key on AAA realm
OneAtlas.sharedInstance()?.authenticateService.login(withAPIKey: "<your_api_key>", clientID: EAAAClientID, block: { (aError) in
    // check errors and continue
    if let error = aError {
        // handle error
    }
    else {
        // retrieve the ID of Living Library Workspace
        OneAtlas.sharedInstance()?.dataService.getMeWith({ (aUser, aError) in
            if let error = aError {
                // handle error
            }
            else {
                // create a filter based on a location
                let location = OAPoint(fromLatitude: 43.0, longitude: 3.0)
                let filter = OASearchFilter()
                filter.setGeometry(location)
                
                let sort = OASort()
                sort.addSortingCriteria(ESortAcquisitionDate, order: ESortOrderDescending)
                
                // perform a search using those criterias
                let page = OAPagination.makePage(0, itemsPerPage: 10)
                OneAtlas.sharedInstance()?.searchService.openSearch(with: page, workspaceID: aUser?.contract.workspaceID, filter: filter, sort: sort, block: { (aFeatures, aPagination, aError) in
                    if let error = aError {
                        // handle error
                    }
                    else {
                        // handle your search results
                    }
                })
            }
        })
    }
})
```


## Sample code: Authenticate

The first step to accomplish before using our SDK is authenticating with our servers.

Please note that all the returns blocks in our APIs will be executed on the main thread, so you shouldn't worry about that before updating your UI.

#### Objective-C
```objective_c
[[[OneAtlas sharedInstance] authenticateService] loginWithAPIKey:@"<your_api_key>" clientID:EAAAClientID block:^(OAError * aError) {
  // check for errors and continue
}];
```

#### Swift 4
```swift
// Swift
OneAtlas.sharedInstance()?.authenticateService.login(withAPIKey: "<your_api_key>", clientID: EAAAClientID, block: { (aError) in
    // check for errors and continue
})
```


## Sample code: Search into a Workspace

This example searches for a set of OAFeature objects, using specific criterias and sorting options.

The search below is performed into the Living Library, then into your own Workspace.

#### Objective-C
```objective_c
// create a filter based on a location
OAPoint * location = [OAPoint pointFromLatitude:43.0 longitude:3.0];
OASearchFilter * filter = [OASearchFilter new];
[filter setGeometry:location];

// create a sort object
OASort * sort = [OASort new];
[sort addSortingCriteria:ESortAcquisitionDate order:ESortOrderDescending];

// create a pagination object
OAPagination * page = [OAPagination makePage:0 itemsPerPage:10];

// perform search in the Living Library
[[[OneAtlas sharedInstance] searchService] openSearchWithPagination:page workspaceID:[OneAtlas livingLibraryWorkspaceID] filter:filter sort:sort block:^(NSArray<OAFeature *> *aFeatures, OAPagination *aPagination, OAError *aError) {
    // check errors and continue
}];


// retrieve your own Workspace ID, and search into it
[[[OneAtlas sharedInstance] dataService] getMeWithBlock:^(OAUser *aUser, OAError *aError) {
    if(aError) {
        // handle error
    }
    else {
        // perform search in your own Workspace
        [[[OneAtlas sharedInstance] searchService] openSearchWithPagination:page workspaceID:aUser.contract.workspaceID filter:filter sort:sort block:^(NSArray<OAFeature *> *aFeatures, OAPagination *aPagination, OAError *aError) {
            // check errors and continue
        }];
    }
}];
```

#### Swift 4
```swift
// create a filter based on a location
let location = OAPoint(fromLatitude: 43.0, longitude: 3.0)
let filter = OASearchFilter()
filter.setGeometry(location)

// create a sort object
let sort = OASort()
sort.addSortingCriteria(ESortAcquisitionDate, order: ESortOrderDescending)

// create a pagination object
let page = OAPagination.makePage(0, itemsPerPage: 10)

// perform search in the Living Library
OneAtlas.sharedInstance()?.searchService.openSearch(with: page, workspaceID:OneAtlas.livingLibraryWorkspaceID(), filter: filter, sort: sort
    , block: { (aFeatures, aPagination, aError) in
        // check errors and continue
})

OneAtlas.sharedInstance()?.dataService.getMeWith({ (aUser, aError) in
    if let error = aError {
        // handle error
    }
    else {
        OneAtlas.sharedInstance()?.searchService.openSearch(with: page, workspaceID: aUser?.contract.workspaceID, filter: filter, sort: sort, block: { (aFeatures, aPagination, aError) in
            // check errors and continue
        })
    }
})
```


## Sample code: Create a complex filter

This sample shows how to enqueue several search criterias into an OASearchFilter object.

#### Objective-C
```objective_c
    // create a complex filter with multiple criterias
    OAPoint * location = [OAPoint pointFromLatitude:43.0 longitude:3.0];
    OASearchFilter * filter = [OASearchFilter new];
    [filter setGeometry:location];
    [filter setResolution:0.5];
    [filter setMinIncidenceAngle:10 included:YES];
    [filter setMaxCloudCover:20 included:NO];
    [filter addConstellation:EConstellationPleiades];
```

#### Swift 4
```swift
// create a complex filter with multiple criterias
let location = OAPoint(fromLatitude: 43.0, longitude: 3.0)
let filter = OASearchFilter()
filter.setGeometry(location)
filter.setResolution(0.5)
filter.setMinIncidenceAngle(10, included: true)
filter.setMaxCloudCover(20, included: false)
filter.add(EConstellationPleiades)
```


## Sample code: Retrieve OneLive WMTS URL and Capabilities

This sample retrieves the OneLive template URL, which you can inject into your favorite mapping engine (provided the latter is compatible with the WMTS standard).

#### Objective-C
```objective_c
[[[OneAtlas sharedInstance] dataService] getOneLiveCapabilitiesWithBlock:^(OACapabilities * aCapabilities,
                                                                           OAError * aError) {
    if(aError) {
        // handle error
    }
    else {
        // get the WMTS template URL
        NSString * onelive_url_template = [aCapabilities wmtsTemplateURLWithTileMatrixSet:@"3857" layerID:@"onelive" useXYZ:YES];
    }
}];
```

#### Swift 4
```swift
OneAtlas.sharedInstance()?.dataService.getOneLiveCapabilities({ (aCapabilities, aError) in
    if let error = aError {
        // handle error
    }
    else {
        // get the WMTS template URL
        let onelive_url_template = aCapabilities?.wmtsTemplateURL(withTileMatrixSet: "3857", layerID: "onelive", useXYZ: true)
    }
})
```

## Sample code: ask for a quotation, and order a product

This sample creates a sample product with a given area, shows how to get a price quotation for its order, then performs the ordering of the selected product.

Please note that this will decrement your credits.

#### Objective-C
```objective_c
// create a basic product order request
OAProduct * product = [OAProduct new];
product.ident = @"<my_feature_id>";
product.aoi = [OAPolygon polygonFromCenter:CLLocationCoordinate2DMake(43.0, 3.0) span:0.05];
product.productType = kProductTypeBundle;
product.radiometricProcessing = kRadiometricProcessingReflectance;
product.imageFormat = kImageFormatJP2;
product.crsCode = kCRSCodeWebMercator;

OAOrderProductRequest * opr = [OAOrderProductRequest orderProductRequestWithProduct:product];
[[[OneAtlas sharedInstance] dataService] getPriceWithRequest:opr block:^(OAPrice *aPrice, OAError *aError) {
    if(aError) {
        // handle error
    }
    else {
        // display price and area
        OAProductPrice * pp = (OAProductPrice *)aPrice;
        NSLog(@"Cost is: %ld", pp.price);
        NSLog(@"Area is: %f", pp.areaKm2);
        
        // order your product
        [[[OneAtlas sharedInstance] dataService] orderProductWithRequest:opr block:^(OAOrder *aOrder, OAError *aError) {
            if(aError) {
                // handle error
            }
            else {
                // ...
            }
        }];
    }
}];
```

#### Swift 4
```swift

// create a basic product order request
let product = OAProduct.init()
product.ident = "<my_feature_id>";
product.aoi = OAPolygon.init(fromCenter: CLLocationCoordinate2DMake(43.0, 3.0), span: 0.05)
product.productType = kProductTypeBundle;
product.radiometricProcessing = kRadiometricProcessingReflectance;
product.imageFormat = kImageFormatJP2;
product.crsCode = kCRSCodeWebMercator;

let opr = OAOrderProductRequest.init(product: product)
OneAtlas.sharedInstance()?.dataService.getProductPrice(with: opr, block: { (aPrice, aError) in
    if let error = aError {
        // handle error
    }
    else {
        // display price and area
        let pp = aPrice as! OAProductPrice
        print("Cost is: \(pp.price)")
        print("Area is: \(pp.areaKm2)")
        
        // order your product
        OneAtlas.sharedInstance()?.dataService.orderProduct(with: opr, block: { (aOrder, aError) in
            if let error = aError {
                // handle error
            }
            else {
                // ...
            }
        })
    }
})
```


## Author

Airbus DS / intelligence-airbus-ds-mobile@airbus.com

## License

Copyright ©2019 Airbus DS.
OneAtlas SDK is available under a proprietary license. See the LICENSE file for more info.

