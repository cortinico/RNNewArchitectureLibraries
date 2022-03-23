# Run

This doc contains the logs of the steps done to achieve the final result.

## Steps

### [[Setup] Create the example-component folder and the package.json]()

1. `mkdir example-library`
1. `touch example-library/package.json`
1. Paste the following code into the `package.json` file
```json
{
    "name": "example-component",
    "version": "0.0.1",
    "description": "Showcase Fabric component with backward compatibility",
    "react-native": "src/index",
    "source": "src/index",
    "files": [
        "src",
        "android",
        "ios",
        "example-component.podspec",
        "!android/build",
        "!ios/build",
        "!**/__tests__",
        "!**/__fixtures__",
        "!**/__mocks__"
    ],
    "keywords": ["react-native", "ios", "android"],
    "repository": "https://github.com/<your_github_handle>/example-component",
    "author": "<Your Name> <your_email@your_provider.com> (https://github.com/<your_github_handle>)",
    "license": "MIT",
    "bugs": {
        "url": "https://github.com/<your_github_handle>/example-component/issues"
    },
    "homepage": "https://github.com/<your_github_handle>/example-component#readme",
    "devDependencies": {},
    "peerDependencies": {
        "react": "*",
        "react-native": "*"
    }
}
```

### [[Native Component] Create the JS import]()

1. `mkdir example-component/src`
1. `touch example-component/src/index.js`
1. Paste the following content into the `index.js`
```js
// @flow

import { requireNativeComponent } from 'react-native'

export default requireNativeComponent("ColoredView")
```

### [[Native Component] Create the iOS implementation]()

1. `mkdir example-component/ios`
1. Open Xcode
1. Create a new static library in the `ios` folder called `RNColoredView`. Keep Objective-C as language.
1. Make that the `Create Git repository on my mac` option is **unchecked**
1. Open finder and arrange the files and folder as shown below:
    ```
    example-library
    '-> ios
        '-> RNColoredView
            '-> RNColoredView.h
            '-> RNColoredView.m
        '-> RNColoredView.xcodeproj
    ```
    It is important that the `RNColoredView.xcodeproj` is a direct child of the `example-component/ios` folder.
1. Remove the `RNColoredView.h`
1. Rename the `RNColoredView.m` into `RNColoredViewManager.m`
1. Replace the code of `RNColoredViewManager.m` with the following
    ```objective-c
    #import <React/RCTViewManager.h>

    @interface RNColoredViewManager : RCTViewManager
    @end

    @implementation RNColoredViewManager

    RCT_EXPORT_MODULE(ColoredView)

    - (UIView *)view
    {
    return [[UIView alloc] init];
    }

    RCT_CUSTOM_VIEW_PROPERTY(color, NSString, UIView)
    {
    [view setBackgroundColor:[self hexStringToColor:json]];
    }

    - hexStringToColor:(NSString *)stringToConvert
    {
    NSString *noHashString = [stringToConvert stringByReplacingOccurrencesOfString:@"#" withString:@""];
    NSScanner *stringScanner = [NSScanner scannerWithString:noHashString];

    unsigned hex;
    if (![stringScanner scanHexInt:&hex]) return nil;
    int r = (hex >> 16) & 0xFF;
    int g = (hex >> 8) & 0xFF;
    int b = (hex) & 0xFF;

    return [UIColor colorWithRed:r / 255.0f green:g / 255.0f blue:b / 255.0f alpha:1.0f];
    }

    @end
    ```
1. In the `example-component` folder, create a `example-component.podspec` file
1. Copy this code in the `podspec` file
```ruby
require "json"

package = JSON.parse(File.read(File.join(__dir__, "package.json")))

Pod::Spec.new do |s|
  s.name            = "example-component"
  s.version         = package["version"]
  s.summary         = package["description"]
  s.description     = package["description"]
  s.homepage        = package["homepage"]
  s.license         = package["license"]
  s.platforms       = { :ios => "11.0" }
  s.author          = package["author"]
  s.source          = { :git => package["repository"], :tag => "#{s.version}" }

  s.source_files    = "ios/**/*.{h,m,mm,swift}"

  s.dependency "React-Core"
end
```

### <a name="test-old-architecture" />[[Native Component] Test The Native Component]()

1. At the same level of example-library run `npx react-native init OldArchitecture`
1. `cd OldArchitecture && yarn add ../example-library`
1. `cd ios && pod install && cd ..`
1. `npx react-native start` (In another terminal, to run Metro)
1. `npx react-native run-ios`
1. Open `OldArchitecture/App.js` file and replace the content with:
    ```js
    /**
     * Sample React Native App
    * https://github.com/facebook/react-native
    *
    * @format
    * @flow strict-local
    */

    import React from 'react';
    import type {Node} from 'react';
    import {
        SafeAreaView,
        StatusBar,
        Text,
        View,
    } from 'react-native';

    import ColoredView from 'example-component/src/index'

    const App: () => Node = () => {

    return (
        <SafeAreaView>
        <StatusBar barStyle={'dark-content'} />
        <ColoredView color="#FF0099" style={{marginLeft:10, marginTop:20, width:100, height:100}}/>
        </SafeAreaView>
        );
    };

    export default App;
    ```
1. Play with the `color` property to see the View background color change

**Note:** OldArchitecture app has not been committed not to pollute the repository.

### [[Fabric Component] Add the JavaScript specs]()

1. `touch example-component/src/ColoredViewNativeComponent.js`
1. Paste the following code:
    ```ts
    // @flow
    import type {ViewProps} from 'react-native/Libraries/Components/View/ViewPropTypes';
    import type {HostComponent} from 'react-native';
    import { ViewStyle } from 'react-native';
    import codegenNativeComponent from 'react-native/Libraries/Utilities/codegenNativeComponent';

    type NativeProps = $ReadOnly<{|
    ...ViewProps,
    color: string
    |}>;

    export default (codegenNativeComponent<NativeProps>(
        'ColoredView',
    ): HostComponent<NativeProps>);
    ```
