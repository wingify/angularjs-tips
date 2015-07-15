## Tips & Tricks

#### 1. Running a code when you aren't sure whether it will be called from within angular context or outside it.

```javascript
$scope.evalAsync(function () {
    // Your code here.
});
```

Note: In case you are sure you are out of angular context, its better to call `$scope.$digest` after your code instead of using `$scope.$apply` because `$apply` will call `$digest` on `$rootScope` which is much heavier.

#### 2. Check if a form is dirty or invalid from your controller.

```javascript
// `formName` is value of name attribute of the form.
// If form is dirty.
if ($scope.formName.$dirty) {
}

// If form is invalid.
if ($scope.formName.$invalid) {
}
```

Note: The name attribute value of the form cannot be hyphenated as that becomes a JS variable.

#### 3. Watching for changes in `name` of any user in a list of users:

```javascript
$scope.watchCollection(
	function watchFn() {
		return $scope.users.map(function (user) {
			return user.name;
		});
	}),
	function watchListener() {
		// Fired when name of any user changes.
	}
});
```


#### 4. Make a particular HTML section compile only after some asynchronous data is fetched:

In Controller:
```javascript
$http.get('data_endpoint').then(function () {
	$scope.isDataFetched = true;
})
```

In HTML:
```html
<div ng-if="isDataFetched">
	<!-- Some complex HTML that needs fetched data -->
</div>
```


#### 5. Angular `copy` function copies the prototype properties of an object on the new object. BEWARE: If you watch such an object, you will probably get stuck in an infinite loop because `$digest` will always find the old and new objects different due to the abnormal behavior of `angular.copy`.

```javascript
var obj = {a: 1};
var obj1 = Object.create(obj);

obj1.hasOwnProperty('a'); // Returns false

var newCopy = angular.copy(obj1);
newCopy.hasOwnProperty('a'); // Returns true. Property `a` is own the object itself now.

```


## Performance:

#### 1. Use track by in ng-repeat expression to avoid re-rendering of compelete list:

```html
<!-- Normal -->
<li ng-repeat="item in items"></li>

<!-- Better -->
<li ng-repeat="item in items track by item.id"></li>

```

#### 2. Debounce the change in model to prevent watchers from firing too frequently (Angular 1.3):

```html
<input type="text" name="userName"
       ng-model="user.name"
       ng-model-options="{ debounce: 300 }" /><br />
```
This will change model value only after staying idle for 300ms after the last change happened in input value.


#### 3. If your page has too many watchers being used to display content which doesn't change with time, use [bindonce](https://github.com/Pasvaz/bindonce)
.

Adds one watcher for `user.name`:

```
Welcome to the app, {{user.name}}
```

Using bindonce, no watcher remains once `user.name` is available:

```html
Welcome to the app, <span bo-text="user.name"></span>
```

Note: Angular 1.3 has a similar feature:

#### 4. If you have a directive that is inside an ng-repeat, its `compile` function is called only once throughout the ng-repeat. So place your common code for directives in compile function so that it executes only once per ng-repeat loop.


#### 5. Watching for an item change in a list:

Bad - Watch with deep comparison creates copies of objects:
```javascript
$scope.watch('yourList',
	function watchListener() {
		// Fired when name of any user changes.
	}
}, true);
```

Good:
```javascript
$scope.watchCollection('yourList',
	function watchListener() {
		// Fired when name of any user changes.
	}
});
```

#### 6. Avoid using filters on large arrays. Filters run on each digest loop and creates new array everytime it runs. In such cases its better to manually watch your list and do filtering when a change is detected.

Bad - Filter runs on each digest loop.

```html
<li ng-repeat="item in items | myComplexSortFilter"></li>
```

Good:

In Controller:
```javascript
$scope.filteredItems = $scope.items;
$scope.$watchCollection('items', function () {
    // Apply filter on change detection
    $scope.filteredItems = myComplexFilter($scope.items);
});
```

In HTML:
```html
<li ng-repeat="item in newitemsList | myComplexSortFilter"></li>
```

## Debugging:

#### 1. Many a times you want to know when an object's particular property gets set. Here is a neat trick to do it:

```javascript
var obj = {a: 2};

var _a = obj.a;

Object.defineProperty(a, 'a', {
 get: function () { return _a; },
 set: function (val) { _a = val; console.log('prop changed'); }
});

// You'll get a console log whenever the property changes.
```

### Contributing
- - -

If you have some Angular tips and tricks you would like to get added here, please open pull request for the same.


### Credits
- - -

The awesome front-end folks at ![Wingify](http://wingify.com/images/logo_wingify_big.png)
