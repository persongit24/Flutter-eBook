<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# 

---

# Chapter 5: State Management in Flutter - Taming the Data Beast

Welcome to perhaps the most crucial chapter in your Flutter journey! If the previous chapter on widgets was about the "look" of your app, this chapter is all about the "brains." State management is often where Flutter developers face their first significant challenge. By the end of this chapter, you'll not only understand the concept of state in Flutter applications but also know how to choose and implement the right state management solution for your projects. Trust me, mastering this topic will level up your Flutter skills dramatically.

## The State Management Problem: Why We Need a Solution

Let's start with a scenario most of us have faced: you've built a beautiful UI with Flutter's widget system, but now you need to share data between different screens, respond to user inputs, and keep your UI in sync with your data. Suddenly, passing data between widgets becomes messy, your code starts to look like spaghetti, and you find yourself asking, "There has to be a better way, right?"

The good news is that there is, but first, let's understand what "state" actually means in a Flutter application.

State, in its simplest form, is any data that can change during the lifetime of your app. This could be user input, authentication status, items in a shopping cart, or even which screen is currently displayed. In Flutter, we typically divide state into two categories:

**Ephemeral (Local) State**: This is state that belongs to a single widget and doesn't need to be shared. A text field's content, whether a checkbox is checked, or an animation's current value are all examples of ephemeral state. This type of state is perfectly managed using StatefulWidget and setState()[^1].

**App (Shared) State**: This is state that's shared across many parts of your application. User profile information, shopping cart contents, or notification settings are examples of app state. This is where dedicated state management solutions become necessary[^1].

The challenge arises when you need to share state between widgets that are far apart in the widget tree. For example, imagine an e-commerce app where adding an item to the cart in one screen should update the cart icon in your app bar and the cart total on another screen. Passing this state up and down the widget tree quickly becomes impractical.

## Flutter's Built-in State Management Options

Before diving into third-party state management solutions, let's understand what Flutter provides out of the box.

### StatefulWidget and setState()

The most basic form of state management in Flutter is through StatefulWidget and its setState() method. This is perfect for managing ephemeral state:

```dart
class CounterPage extends StatefulWidget {
  @override
  _CounterPageState createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Counter')),
      body: Center(
        child: Text('Counter value: $_counter', style: TextStyle(fontSize: 24)),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

When the FloatingActionButton is pressed, _incrementCounter() is called, which updates the _counter variable and calls setState(). This tells Flutter to rebuild the widget with the new state. It's simple and works well for localized state[^5].

However, setState() has limitations:

1. It rebuilds the entire widget, which can be inefficient for complex UIs
2. It doesn't scale well for state that needs to be shared across multiple widgets
3. It tightly couples your UI and business logic[^6]

### InheritedWidget

A more advanced built-in option is InheritedWidget, which allows you to efficiently pass data down the widget tree without manually passing it through each level:

```dart
class DataProvider extends InheritedWidget {
  final int data;
  
  DataProvider({required this.data, required Widget child})
      : super(child: child);
  
  static DataProvider? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<DataProvider>();
  }
  
  @override
  bool updateShouldNotify(DataProvider oldWidget) {
    return data != oldWidget.data;
  }
}

// Usage
DataProvider(
  data: 42,
  child: MyWidget(),
)

// In a descendant widget
int data = DataProvider.of(context)!.data;
```

InheritedWidget is powerful and efficient, but writing all the boilerplate code can be tedious. It's also more about providing access to data than managing how that data changes[^7].

## Popular State Management Solutions

Now let's explore some of the most popular state management solutions in the Flutter ecosystem. Remember, there's no "one-size-fits-all" solution – each has its strengths and ideal use cases.

### Provider

Provider is a wrapper around InheritedWidget that makes it significantly easier to use. It's simple, lightweight, and recommended by the Flutter team for most applications[^2].

Here's how it works:

```dart
// 1. Create a model class
class CounterModel extends ChangeNotifier {
  int _count = 0;
  int get count => _count;
  
  void increment() {
    _count++;
    notifyListeners();
  }
}

// 2. Provide the model to your widget tree
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CounterModel(),
      child: MyApp(),
    ),
  );
}

// 3. Consume the model in your widgets
class CounterDisplay extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Text(
      'Count: ${context.watch<CounterModel>().count}',
      style: TextStyle(fontSize: 24),
    );
  }
}

class CounterIncrementer extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return FloatingActionButton(
      onPressed: () => context.read<CounterModel>().increment(),
      child: Icon(Icons.add),
    );
  }
}
```

Provider uses the concept of "providers" (data sources) and "consumers" (widgets that need the data). When the data changes, only the consumers of that particular data are rebuilt, which is much more efficient than rebuilding the entire widget tree[^2].

### Riverpod

Riverpod is often described as "Provider, but better." Created by the same author as Provider, Riverpod addresses some of Provider's limitations while maintaining its simplicity:

```dart
// 1. Define providers
final counterProvider = StateNotifierProvider<CounterNotifier, int>((ref) {
  return CounterNotifier();
});

class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);
  
  void increment() => state++;
}

// 2. Consume the provider
class CounterWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    
    return Scaffold(
      appBar: AppBar(title: Text('Riverpod Counter')),
      body: Center(
        child: Text('Count: $count', style: TextStyle(fontSize: 24)),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => ref.read(counterProvider.notifier).increment(),
        child: Icon(Icons.add),
      ),
    );
  }
}
```

Riverpod's advantages include compile-time safety (no more runtime provider not found errors), the ability to easily handle async data, and better testing capabilities[^4].

### BLoC (Business Logic Component)

BLoC is a state management pattern that uses streams and separates your application into three layers:

1. **UI Layer**: Widgets that display data and send events
2. **BLoC Layer**: Business logic that processes events and outputs states
3. **Data Layer**: Repositories and models that handle data

Here's a simplified example:

```dart
// 1. Define events and states
abstract class CounterEvent {}
class IncrementEvent extends CounterEvent {}

// 2. Create a BLoC
class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<IncrementEvent>((event, emit) {
      emit(state + 1);
    });
  }
}

// 3. Provide and consume the BLoC
class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => CounterBloc(),
      child: CounterView(),
    );
  }
}

class CounterView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('BLoC Counter')),
      body: Center(
        child: BlocBuilder<CounterBloc, int>(
          builder: (context, count) {
            return Text('Count: $count', style: TextStyle(fontSize: 24));
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => context.read<CounterBloc>().add(IncrementEvent()),
        child: Icon(Icons.add),
      ),
    );
  }
}
```

BLoC is excellent for complex applications with lots of business logic. It enforces a clean separation of concerns and is highly testable. However, it has a steeper learning curve and requires more boilerplate code than Provider or Riverpod[^3].

### GetX

GetX is a lightweight yet powerful solution that combines state management, dependency injection, and route management:

```dart
// 1. Create a controller
class CounterController extends GetxController {
  var count = 0.obs;
  
  void increment() => count++;
}

// 2. Use the controller in your UI
class CounterPage extends StatelessWidget {
  final CounterController controller = Get.put(CounterController());
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('GetX Counter')),
      body: Center(
        child: Obx(() => Text(
          'Count: ${controller.count.value}',
          style: TextStyle(fontSize: 24),
        )),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: controller.increment,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

GetX uses reactive programming principles with minimal boilerplate. It's particularly popular for its simplicity and the extra utilities it provides beyond just state management[^11][^14].

### MobX

MobX is a battle-tested state management library ported from React to Flutter:

```dart
// 1. Create a store
class Counter = _Counter with _$Counter;

abstract class _Counter with Store {
  @observable
  int value = 0;
  
  @action
  void increment() {
    value++;
  }
}

// 2. Use the store in your UI
class CounterPage extends StatelessWidget {
  final counter = Counter();
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('MobX Counter')),
      body: Center(
        child: Observer(
          builder: (_) => Text(
            'Count: ${counter.value}',
            style: TextStyle(fontSize: 24),
          ),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: counter.increment,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

MobX uses the concepts of observables, actions, and reactions to manage state. It's particularly good at handling complex object graphs and interdependent state[^9][^12][^15].

### Redux

Redux is another state management pattern borrowed from the React ecosystem:

```dart
// 1. Define actions and reducer
enum Actions { Increment }

int counterReducer(int state, dynamic action) {
  if (action == Actions.Increment) {
    return state + 1;
  }
  return state;
}

// 2. Create a store
final store = Store<int>(counterReducer, initialState: 0);

// 3. Connect your UI to the store
class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return StoreProvider<int>(
      store: store,
      child: Scaffold(
        appBar: AppBar(title: Text('Redux Counter')),
        body: Center(
          child: StoreConnector<int, String>(
            converter: (store) => store.state.toString(),
            builder: (context, count) {
              return Text('Count: $count', style: TextStyle(fontSize: 24));
            },
          ),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => store.dispatch(Actions.Increment),
          child: Icon(Icons.add),
        ),
      ),
    );
  }
}
```

Redux enforces a unidirectional data flow and centralized state, which can be beneficial for applications with complex state logic[^10][^13][^16].

## Choosing the Right Solution for Your App

With so many options, how do you choose the right state management solution? Here are some guidelines:

1. **For simple apps**: Use setState() for truly local state, and Provider for shared state.
2. **For medium-sized apps**: Consider Riverpod or GetX, which offer a good balance of simplicity and power.
3. **For complex apps**: BLoC, MobX, or Redux provide more structure and scalability, though with steeper learning curves.

Also consider these factors:

- **Team experience**: If your team knows React-Redux, Flutter-Redux might be easier to adopt.
- **Specific requirements**: Need dependency injection too? GetX might be ideal.
- **Testing needs**: BLoC and Riverpod are particularly test-friendly.

Remember, you can mix and match solutions! Use setState() for simple local state while using another solution for app-wide state.

## Implementing a Real-World Example: Shopping Cart

To tie everything together, let's implement a shopping cart feature using Provider as our state management solution. This will show you how to manage more complex state in a real-world scenario.

First, let's define our models:

```dart
class Product {
  final String id;
  final String name;
  final double price;
  final String imageUrl;
  
  Product({
    required this.id, 
    required this.name, 
    required this.price, 
    required this.imageUrl
  });
}

class CartItem {
  final Product product;
  int quantity;
  
  CartItem({required this.product, this.quantity = 1});
  
  double get totalPrice => product.price * quantity;
}
```

Next, let's create our cart model:

```dart
class CartModel extends ChangeNotifier {
  final List<CartItem> _items = [];
  
  List<CartItem> get items => List.unmodifiable(_items);
  
  int get itemCount => _items.fold(0, (sum, item) => sum + item.quantity);
  
  double get totalPrice => _items.fold(
    0, (sum, item) => sum + item.totalPrice
  );
  
  void addProduct(Product product) {
    final existingItemIndex = _items.indexWhere(
      (item) => item.product.id == product.id
    );
    
    if (existingItemIndex >= 0) {
      _items[existingItemIndex].quantity++;
    } else {
      _items.add(CartItem(product: product));
    }
    
    notifyListeners();
  }
  
  void removeItem(String productId) {
    _items.removeWhere((item) => item.product.id == productId);
    notifyListeners();
  }
  
  void incrementQuantity(String productId) {
    final item = _items.firstWhere((item) => item.product.id == productId);
    item.quantity++;
    notifyListeners();
  }
  
  void decrementQuantity(String productId) {
    final item = _items.firstWhere((item) => item.product.id == productId);
    if (item.quantity > 1) {
      item.quantity--;
    } else {
      _items.remove(item);
    }
    notifyListeners();
  }
  
  void clearCart() {
    _items.clear();
    notifyListeners();
  }
}
```

Now, let's create our product listing page:

```dart
class ProductListingPage extends StatelessWidget {
  final List<Product> products = [
    Product(
      id: '1', 
      name: 'Sneakers', 
      price: 99.99, 
      imageUrl: 'assets/sneakers.jpg'
    ),
    Product(
      id: '2', 
      name: 'Backpack', 
      price: 49.99, 
      imageUrl: 'assets/backpack.jpg'
    ),
    // Add more products...
  ];
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Shop'),
        actions: [
          Consumer<CartModel>(
            builder: (context, cart, child) {
              return Badge(
                label: Text('${cart.itemCount}'),
                child: IconButton(
                  icon: Icon(Icons.shopping_cart),
                  onPressed: () {
                    Navigator.of(context).push(
                      MaterialPageRoute(
                        builder: (context) => CartPage(),
                      ),
                    );
                  },
                ),
              );
            },
          ),
        ],
      ),
      body: GridView.builder(
        padding: EdgeInsets.all(16),
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          childAspectRatio: 2/3,
          crossAxisSpacing: 10,
          mainAxisSpacing: 10,
        ),
        itemCount: products.length,
        itemBuilder: (context, index) {
          return ProductCard(product: products[index]);
        },
      ),
    );
  }
}

class ProductCard extends StatelessWidget {
  final Product product;
  
  const ProductCard({required this.product});
  
  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 2,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Expanded(
            child: Container(
              width: double.infinity,
              child: Image.asset(
                product.imageUrl,
                fit: BoxFit.cover,
              ),
            ),
          ),
          Padding(
            padding: EdgeInsets.all(8),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  product.name,
                  style: TextStyle(
                    fontSize: 16,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                SizedBox(height: 4),
                Text(
                  '\$${product.price.toStringAsFixed(2)}',
                  style: TextStyle(
                    fontSize: 14,
                    color: Colors.grey[^700],
                  ),
                ),
                SizedBox(height: 8),
                Consumer<CartModel>(
                  builder: (context, cart, child) {
                    return ElevatedButton(
                      onPressed: () {
                        cart.addProduct(product);
                        ScaffoldMessenger.of(context).showSnackBar(
                          SnackBar(
                            content: Text('${product.name} added to cart!'),
                            duration: Duration(seconds: 1),
                          ),
                        );
                      },
                      child: Text('Add to Cart'),
                    );
                  },
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

And finally, our cart page:

```dart
class CartPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Shopping Cart'),
      ),
      body: Consumer<CartModel>(
        builder: (context, cart, child) {
          if (cart.items.isEmpty) {
            return Center(
              child: Text(
                'Your cart is empty',
                style: TextStyle(fontSize: 18),
              ),
            );
          }
          
          return Column(
            children: [
              Expanded(
                child: ListView.builder(
                  itemCount: cart.items.length,
                  itemBuilder: (context, index) {
                    final item = cart.items[index];
                    return ListTile(
                      leading: Image.asset(
                        item.product.imageUrl,
                        width: 50,
                        height: 50,
                        fit: BoxFit.cover,
                      ),
                      title: Text(item.product.name),
                      subtitle: Text('\$${item.product.price.toStringAsFixed(2)}'),
                      trailing: Row(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          IconButton(
                            icon: Icon(Icons.remove),
                            onPressed: () {
                              cart.decrementQuantity(item.product.id);
                            },
                          ),
                          Text('${item.quantity}'),
                          IconButton(
                            icon: Icon(Icons.add),
                            onPressed: () {
                              cart.incrementQuantity(item.product.id);
                            },
                          ),
                          IconButton(
                            icon: Icon(Icons.delete),
                            onPressed: () {
                              cart.removeItem(item.product.id);
                            },
                          ),
                        ],
                      ),
                    );
                  },
                ),
              ),
              Container(
                padding: EdgeInsets.all(16),
                decoration: BoxDecoration(
                  color: Colors.white,
                  boxShadow: [
                    BoxShadow(
                      color: Colors.black12,
                      blurRadius: 4,
                      offset: Offset(0, -2),
                    ),
                  ],
                ),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.stretch,
                  children: [
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        Text(
                          'Total:',
                          style: TextStyle(
                            fontSize: 18,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                        Text(
                          '\$${cart.totalPrice.toStringAsFixed(2)}',
                          style: TextStyle(
                            fontSize: 18,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ],
                    ),
                    SizedBox(height: 16),
                    ElevatedButton(
                      style: ElevatedButton.styleFrom(
                        padding: EdgeInsets.symmetric(vertical: 12),
                      ),
                      onPressed: () {
                        // Checkout logic
                        cart.clearCart();
                        Navigator.of(context).pop();
                        ScaffoldMessenger.of(context).showSnackBar(
                          SnackBar(
                            content: Text('Order placed successfully!'),
                          ),
                        );
                      },
                      child: Text(
                        'CHECKOUT',
                        style: TextStyle(fontSize: 16),
                      ),
                    ),
                  ],
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}
```

To use this in your app, you'd wrap your main app with a ChangeNotifierProvider:

```dart
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => CartModel(),
      child: MyApp(),
    ),
  );
}
```

This shopping cart example demonstrates several key state management concepts:

1. **Central state**: The cart's contents are managed in a single CartModel
2. **State updates**: Adding, removing, and updating items triggers UI updates
3. **State access**: Different widgets access the same state (product listing and cart page)
4. **State persistence**: The cart persists as you navigate between screens

## State Management Best Practices

To wrap up this chapter, here are some best practices to follow regardless of which state management solution you choose:

1. **Keep it simple**: Don't over-engineer your state management. Start with the simplest solution that meets your needs and evolve as necessary.
2. **Separate concerns**: Keep your UI logic separate from your business logic. Your widgets should focus on presentation, while your state management handles data and business rules.
3. **Be consistent**: Pick a pattern and stick with it throughout your app. Mixing different state management approaches can lead to confusion and bugs.
4. **Consider performance**: Be mindful of unnecessary rebuilds. Most state management solutions provide ways to optimize which widgets rebuild when state changes.
5. **Think about testing**: Good state management makes your app easier to test. Business logic should be testable independent of your UI.
6. **Use immutable data**: When possible, treat your state as immutable. This prevents unexpected side effects and makes your code more predictable.
7. **Organize your code**: Structure your state management code logically. Group related state together and keep your models clean and focused.
8. **Plan for the future**: Choose a solution that can scale with your app. What works for a small app might not work well as it grows.

## Conclusion

State management is a crucial aspect of Flutter development, and mastering it will significantly improve your ability to build complex, maintainable applications. In this chapter, we've explored various state management solutions, from Flutter's built-in options to popular third-party packages.

Remember, there's no one-size-fits-all solution. The best state management approach depends on your app's specific needs, your team's experience, and your personal preferences. Don't be afraid to experiment with different solutions to find what works best for you.

In the next chapter, we'll dive into navigation and routing in Flutter, exploring how to create multi-screen applications with smooth transitions between screens. We'll also look at how state management interacts with navigation to create cohesive user experiences.

Before we move on, I'd love to hear about your experience with state management. Have you tried any of the solutions we discussed? Do you have questions about implementing state management in your own projects? Your input will help shape the content of our upcoming chapters, ensuring they address your specific needs and interests.

<div style="text-align: center">⁂</div>

[^1]: https://docs.flutter.dev/get-started/fundamentals/state-management

[^2]: https://www.youtube.com/watch?v=CvRkyxJn2Fc

[^3]: https://www.dhiwise.com/post/flutter-bloc-tutorial-understanding-state-management

[^4]: https://riverpod.dev

[^5]: https://themobilecoder.com/flutter-setstate-the-simplest-state-management-in-flutter/

[^6]: https://theonetechnologies.com/blog/post/mastering-state-management-in-flutter-a-comprehensive-guide

[^7]: https://www.kodeco.com/39848254-the-top-5-flutter-state-management-solutions-a-deep-dive

[^8]: https://www.techaheadcorp.com/blog/master-state-management-in-flutter-best-practices-and-patterns/

[^9]: https://circleci.com/blog/state-management-for-flutter-apps-with-mobx/

[^10]: https://flutterexperts.com/redux-state-management-in-flutter/

[^11]: https://www.dhiwise.com/post/understanding-flutter-getx-state-manager

[^12]: https://mobx.netlify.app

[^13]: https://medium.flutterdevs.com/redux-state-management-in-flutter-a64873dec1d0

[^14]: https://www.bacancytechnology.com/blog/flutter-state-management-using-getx

[^15]: https://30dayscoding.com/blog/building-flutter-apps-with-mobx-state-management

[^16]: https://www.dhiwise.com/post/the-ultimate-guide-to-state-management-with-flutter-redux

[^17]: https://www.dhiwise.com/post/top-flutter-state-management-packages-2023

[^18]: https://www.youtube.com/watch?v=FUDhozpnTUw

[^19]: https://www.flutterclutter.dev/flutter/basics/what-is-the-bloc-pattern/2021/2084/

[^20]: https://codewithandrea.com/articles/flutter-state-management-riverpod/

[^21]: https://raman04.hashnode.dev/using-setstate-for-simple-state-management-in-flutter

[^22]: https://www.icoderzsolutions.com/blog/flutter-state-management-packages/

[^23]: https://widgettricks.substack.com/p/state-management

[^24]: https://code.zeba.academy/advanced-state-management-flutter-provider-bloc-riverpod-patterns/

[^25]: https://www.social.plus/tutorials/state-management-in-dart-flutter

[^26]: https://reliasoftware.com/blog/provider-in-flutter-for-state-management

[^27]: https://pub.dev/packages/flutter_bloc

[^28]: https://stackoverflow.com/questions/78138989/flutter-riverpod-should-i-use-a-riverpod-statenotifier-provider-to-represent-f

[^29]: https://github.com/jonataslaw/getx/blob/master/documentation/en_US/state_management.md

[^30]: https://www.dhiwise.com/post/mobx-state-management-in-flutter-effective-library-in-action

[^31]: https://blog.logrocket.com/flutter-redux-complete-tutorial-with-examples/

[^32]: https://pub.dev/packages/get

[^33]: https://pub.dev/packages/mobx

[^34]: https://pub.dev/documentation/flutter_redux/latest/

[^35]: https://blog.logrocket.com/ultimate-guide-getx-state-management-flutter/

[^36]: https://docs.flutter.dev/data-and-backend/state-mgmt/options

[^37]: https://www.youtube.com/watch?v=60_2HlagOzg

[^38]: https://www.youtube.com/watch?v=QJ314HaiZ1g

[^39]: https://www.reddit.com/r/FlutterDev/comments/1dwiu95/struggling_with_state_management_in_flutter_which/

[^40]: https://sunscrapers.com/blog/implement-redux-with-flutter-app/

