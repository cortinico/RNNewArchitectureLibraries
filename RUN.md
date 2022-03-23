# Run

This doc contains the logs of the steps done to achieve the final result.

## Steps

### [[Setup] Create the example-library folder and the package.json](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/0f64d565f214c2848b475e743834da5751abc8e0)

1. `mkdir example-library`
1. `touch example-library/package.json`
1. Paste the following code into the `package.json` file
```json
{
    "name": "example-library",
    "version": "0.0.1",
    "description": "Showcase Turbomodule with backward compatibility",
    "react-native": "src/index",
    "source": "src/index",
    "files": [
        "src",
        "android",
        "ios",
        "example-library.podspec",
        "!android/build",
        "!ios/build",
        "!**/__tests__",
        "!**/__fixtures__",
        "!**/__mocks__"
    ],
    "keywords": ["react-native", "ios", "android"],
    "repository": "https://github.com/<your_github_handle>/example-library",
    "author": "<Your Name> <your_email@your_provider.com> (https://github.com/<your_github_handle>)",
    "license": "MIT",
    "bugs": {
        "url": "https://github.com/<your_github_handle>/example-library/issues"
    },
    "homepage": "https://github.com/<your_github_handle>/example-library#readme",
    "devDependencies": {},
    "peerDependencies": {
        "react": "*",
        "react-native": "*"
    }
}
```

### [[Native Module] Create the JS import](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/952ea481f17733037278e2367270fd238097b9d9)


1. `mkdir example-library/src`
1. `touch example-library/src/index.js`
1. Paste the following content into the `index.js`
```js
// @flow
import { NativeModules } from 'react-native'

export default NativeModules.Calculator;
```

### [[Native Module] Create the iOS implementation](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/b2d189c362aeff4dddf36010639ca076969ca831)

1. `mkdir example-library/ios`
1. Open Xcode
1. Create a new static library in the `ios` folder called `RNCalculator`. Keep Objective-C as language.
1. Make that the `Create Git repository on my mac` option is **unchecked**
1. Open finder and arrange the files and folder as shown below:
    ```
    example-library
    '-> ios
        '-> RNCalculator
            '-> RNCalculator.h
            '-> RNCalculator.m
        '-> RNCalculator.xcodeproj
    ```
    It is important that the `RNCalculator.xcodeproj` is a direct child of the `example-library/ios` folder.
1. Open the `RNCalculator.h` file and update the code as it follows:
    ```diff
    - #import <Foundation/Foundation.h>
    + #import <React/RCTBridgeModule.h>

    + @interface RNCalculator : NSObject <RCTBridgeModule>

    @end
    ```
1. Open the `RNCalculator.m` file and replace the code with the following:
    ```objective-c
    #import "RNCalculator.h"

    @implementation RNCalculator

    RCT_EXPORT_MODULE(Calculator)

    RCT_REMAP_METHOD(add, addA:(NSInteger)a
                            andB:(NSInteger)b
                    withResolver:(RCTPromiseResolveBlock) resolve
                    withRejecter:(RCTPromiseRejectBlock) reject)
    {
        NSNumber *result = [[NSNumber alloc] initWithInteger:a+b];
        resolve(result);
    }

    @end
    ```
1. In the `example-library` folder, create a `example-library.podspec` file
1. Copy this code in the `podspec` file
```ruby
require "json"

package = JSON.parse(File.read(File.join(__dir__, "package.json")))

Pod::Spec.new do |s|
  s.name            = "example-library"
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

### <a name="test-old-architecture" />[[Native Module] Test The Native Module](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/3b79ee7639d54f108e33fc8c17dd62d842eba0c9)

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
    import {useState} from "react";
    import type {Node} from 'react';
    import {
    SafeAreaView,
    StatusBar,
    Text,
    Button,
    } from 'react-native';

    import Calculator from 'example-library/src/index'

    const App: () => Node = () => {
    const [currentResult, setResult] = useState<number | null>(null);
    return (
        <SafeAreaView>
        <StatusBar barStyle={'dark-content'}/>
        <Text style={{marginLeft:20, marginTop:20}}>3+7={currentResult ?? "??"}</Text>
        <Button title="Compute" onPress={async () => {
            const result = await Calculator.add(3, 7);
            setResult(result);
        }} />
        </SafeAreaView>
    );
    };

    export default App;
    ```
1. Click on the `Compute` button and see the app working

**Note:** OldArchitecture app has not been committed not to pollute the repository.

### [[TurboModule] Add the JavaScript specs](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/54aedba81461301f4e2bc602ebc8df78e0df2639)

1. `touch example-library/src/NativeCalculator.js`
1. Paste the following code:
    ```ts
    // @flow
    import type { TurboModule } from 'react-native/Libraries/TurboModule/RCTExport';
    import { TurboModuleRegistry } from 'react-native';

    export interface Spec extends TurboModule {
    // your module methods go here, for example:
    add(a: number, b: number): Promise<number>;
    }
    export default (TurboModuleRegistry.get<Spec>(
    'Calculator'
    ): ?Spec);
    ```

### [[TurboModule] Set up CodeGen](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/553984008f10a461e7cf051221e145b310809e8c)

1. Open the `example-library/package.json`
1. Add the following snippet at the end of it:
    ```json
    ,
    "codegenConfig": {
        "libraries": [
            {
            "name": "RNCalculatorSpec",
            "type": "modules",
            "jsSrcsDir": "src"
            }
        ]
    }
    ```

### [[TurboModule] Set up `podspec` file](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/72201a362cffea5c77a1eb322e3f9f202cfcb83c)

1. Open the `example-library/example-library.podspec` file
1. Before the `Pod::Spec.new do |s|` add the following code:
    ```ruby
    folly_version = '2021.06.28.00-v2'
    folly_compiler_flags = '-DFOLLY_NO_CONFIG -DFOLLY_MOBILE=1 -DFOLLY_USE_LIBCPP=1 -Wno-comma -Wno-shorten-64-to-32'
    ```
1. Before the `end ` tag, add the following code
    ```ruby
    # This guard prevent to install the dependencies when we run `pod install` in the old architecture.
    if ENV['RCT_NEW_ARCH_ENABLED'] == '1' then
        s.compiler_flags = folly_compiler_flags + " -DRCT_NEW_ARCH_ENABLED=1"
        s.pod_target_xcconfig    = {
            "HEADER_SEARCH_PATHS" => "\"$(PODS_ROOT)/boost\"",
            "CLANG_CXX_LANGUAGE_STANDARD" => "c++17"
        }

        s.dependency "React-Codegen"
        s.dependency "RCT-Folly", folly_version
        s.dependency "RCTRequired"
        s.dependency "RCTTypeSafety"
        s.dependency "ReactCommon/turbomodule/core"
    end
    ```

### [[TurboModule] Update the Native iOS code](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/be974636395e4459f61f650152ce33b0312759ab)

1. In the `ios/RNCalculator` folder, rename the `RNCalculator.m` into `RNCalculator.mm`
1. Open it and add the following `import`:
    ```c++
    // Thanks to this guard, we won't import this header when we build for the old architecture.
    #ifdef RCT_NEW_ARCH_ENABLED
    #import "RNCalculatorSpec.h"
    #endif
    ```
1. Then, before the `@end` keyword, add the following code:
    ```c++
    // Thanks to this guard, we won't compile this code when we build for the old architecture.
    #ifdef RCT_NEW_ARCH_ENABLED
    - (std::shared_ptr<facebook::react::TurboModule>)getTurboModule:
        (const facebook::react::ObjCTurboModule::InitParams &)params
    {
        return std::make_shared<facebook::react::NativeCalculatorSpecJSI>(params);
    }
    #endif
    ```
### [[TurboModule] Unify JavaScript interface](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/842cd89827f876994f293279282655a653545856)

1. Open the `src/index.js` file
1. Replace the code with the following:
    ```ts
    // @flow
    import { NativeModules } from 'react-native'

    const isTurboModuleEnabled = global.__turboModuleProxy != null;

    const calculator = isTurboModuleEnabled ?
    require("./NativeCalculator").default :
    NativeModules.Calculator;

    export default calculator;
    ```

### [[TurboModule] Test the Turbomodule](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/7c243206fba6ea1a55ffd03478a7f075bd15429d)

1. At the same level of example-library run `npx react-native init NewArchitecture --version next` (`next` takes the next version that is about to be released. Any version >= 0.68 should work)
1. `cd NewArchitecture && yarn add ../example-library`
1. `cd ios && RCT_NEW_ARCH_ENABLED=1 pod install && cd ..`
1. `npx react-native start` (In another terminal, to run Metro)
1. `npx react-native run-ios`
1. Open `NewArchitecture/App.js` file and replace the content with the same file used for the [`OldArchitecture`](#test-old-architecture).
1. Click on the `Compute` button and see the app working

**Note:** OldArchitecture app has not been committed not to pollute the repository.
