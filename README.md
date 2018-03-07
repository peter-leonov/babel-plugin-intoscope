https://astexplorer.net
https://github.com/jamiebuilds/babel-handbook/blob/master/translations/en/plugin-handbook.md#toc-paths
https://babeljs.io/docs/core-packages/babel-types/

Goals:

* pass imported DSs
* allow some imports to be actually imported (using test runner)
* export any top scope ID in a wrapper
* allow module IDs refer to original versions of other module IDs

## Usage

    import scope from './filter';

    test('filter against', () => {
        // provide imported functions/values mocks
        const foo = scope({map: mapMock, get: getMock});
        //
        const filter = foo({})
    });

## Limitations by design

### Module purity

The main limitation is that the module tested using introscope should be a pure module. This means requiring it makes no side effects. The example module is pure:

    import dropDatabase from 'api'
    // this module never calls dropDatabase() itself
    export function safeDropDatabase(password) {
        if (password == '123456')
            dropDatabase();
    }

and the following is not:

    import dropDatabase from 'api'
    // this module calls dropDatabase() each time it gets imported
    if (process.env.node_env == 'test') {
        dropDatabase();
    }

This limitation is easy to avoid putting all the side effects code in an exported function and call it outside of the module in the application code. Anyway, ES6 modules expected to be pure, so making a module testable using Introscope is just another good reason to make all your modules pure.

### Pure imports

Ignored imports are pure. This is an example of an impure import:

    import map from 'lodash'
    // map just maps here
    import 'magic-hack'
    // map launches missles here

In this case both imports should be ignored as `lodash` and `magic-hack` are dependent modules and form a unit together.

In case only some of the imported values has to be ignored ignore them like this:

    // @introscope ignore: ['loadItems']
    import { loadItems, saveItems } from 'api'
    // here `loadItems` comes from 'api' module
    // and `saveItems` comes from a test

### Constant testees

Introscope will warn you if the testee variable is not constant

## Nice tricks

Wrap all possible IDs in a test-plan like proxy, mock imported side effects and then just run each funtion with different input and record how proxies been called and what returned.

In case of a dynamic import value ([bindings](http://2ality.com/2015/07/es6-module-exports.html)) like `ticksCounter` here:

    import { ticksCounter, tick } from 'date'
    console.log(ticksCounter) // 0
    tick()
    console.log(ticksCounter) // 1

this import needs to be wrapped in a `Proxy` to preserve the semantics of ES6 modules.
