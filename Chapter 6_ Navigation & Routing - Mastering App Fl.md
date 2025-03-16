# Chapter 6: Navigation \& Routing - Mastering App Flow

Welcome to one of the most practical chapters in our Flutter journey! Navigation is what transforms a collection of screens into an actual app experience. Think about your favorite apps—whether it's Instagram, Spotify, or Uber—their intuitive navigation is a huge part of what makes them feel polished and professional. By the end of this chapter, you'll have the skills to create that same seamless flow in your own Flutter apps.

## The Navigation Paradigm: More Than Just Screens

Before we dive into the code, let's understand what navigation really means in the context of mobile apps. Navigation isn't just about moving between screens—it's about creating a coherent journey for your users. It's about answering questions like:

- How do users move between different sections of your app?
- How do they go back to previous screens?
- How does your app remember where users were if they close and reopen it?
- How do deep links from outside your app land users in the right place?

Flutter offers two distinct approaches to navigation: the imperative Navigator 1.0 API and the newer, declarative Router API (often called Navigator 2.0). The first is simpler but less powerful, while the second is more complex but offers greater control. Most apps start with Navigator 1.0 and graduate to the Router API as their navigation needs grow more complex.

Let's explore both approaches, starting with the more straightforward one.

## Navigator 1.0: The Stack-Based Approach

At its core, Navigator 1.0 works like a stack of cards. When you navigate to a new screen, you're pushing it onto the top of the stack. When you go back, you're popping the top card off to reveal what's underneath.

This mental model is crucial to understanding Flutter navigation: screens are ordered in a last-in, first-out (LIFO) stack. The screen at the top of the stack is what the user sees.

Let's implement basic navigation between two screens:

```dart
// First Screen
class FirstScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('First Screen')),
      body: Center(
        child: ElevatedButton(
          child: Text('Go to Second Screen'),
          onPressed: () {
            // Navigate to second screen
            Navigator.push(
              context,
              MaterialPageRoute(builder: (context) => SecondScreen()),
            );
          },
        ),
      ),
    );
  }
}

// Second Screen
class SecondScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Second Screen')),
      body: Center(
        child: ElevatedButton(
          child: Text('Go Back'),
          onPressed: () {
            // Navigate back to first screen
            Navigator.pop(context);
          },
        ),
      ),
    );
  }
}
```

When the button on the first screen is pressed, `Navigator.push()` adds the second screen to the top of the stack. When the button on the second screen is pressed, `Navigator.pop()` removes the second screen, revealing the first screen again.

This push-and-pop mechanism is the foundation of Flutter navigation, and it works beautifully for simple navigation flows. Let's look at some more advanced techniques.

### Passing Data Between Screens

In real-world apps, you often need to pass data between screens. For example, you might want to show details of an item selected from a list. Here's how to pass data forward:

```dart
// In FirstScreen
ElevatedButton(
  child: Text('View Details'),
  onPressed: () {
    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (context) => DetailScreen(item: selectedItem),
      ),
    );
  },
)

// DetailScreen
class DetailScreen extends StatelessWidget {
  final Item item;
  
  const DetailScreen({required this.item});
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(item.name)),
      body: // Display item details
    );
  }
}
```

But what about returning data from a screen? For instance, imagine a screen where users select an option, and you need that selection back in the previous screen. Flutter makes this easy with `Navigator.pop()`:

```dart
// In OptionScreen
ElevatedButton(
  child: Text('Select Option A'),
  onPressed: () {
    Navigator.pop(context, 'Option A');
  },
)

// Back in the original screen, when navigating
void _navigateAndGetResult() async {
  final result = await Navigator.push(
    context,
    MaterialPageRoute(builder: (context) => OptionScreen()),
  );
  
  if (result != null) {
    setState(() {
      selectedOption = result;
    });
  }
}
```

The `await` keyword is crucial here—it tells Flutter to pause execution until the user returns from the pushed screen, then continue with the result.

### Named Routes: Organized Navigation

As your app grows, directly pushing widget instances becomes unwieldy. Named routes offer a more organized approach, defining your navigation paths in a central location. Here's how it works:

```dart
// In your MaterialApp
MaterialApp(
  initialRoute: '/',
  routes: {
    '/': (context) => HomeScreen(),
    '/details': (context) => DetailScreen(),
    '/settings': (context) => SettingsScreen(),
    '/profile': (context) => ProfileScreen(),
  },
)

// To navigate
ElevatedButton(
  child: Text('Go to Settings'),
  onPressed: () {
    Navigator.pushNamed(context, '/settings');
  },
)
```

This approach has several advantages:

- All routes are defined in one place, making your navigation structure easier to understand
- You can navigate to routes from anywhere in your app without needing to import specific screen widgets
- It's a stepping stone toward more advanced routing techniques

But what about passing arguments with named routes? Flutter has you covered:

```dart
// Define your route with arguments
Navigator.pushNamed(
  context,
  '/details',
  arguments: {'id': 123, 'title': 'Awesome Product'},
);

// And retrieve them in the destination screen
class DetailScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final args = ModalRoute.of(context)!.settings.arguments as Map<String, dynamic>;
    
    return Scaffold(
      appBar: AppBar(title: Text(args['title'])),
      body: // Use args['id'] to fetch details
    );
  }
}
```

For more complex routing needs, you can use the `onGenerateRoute` property in `MaterialApp` to handle dynamic routes:

```dart
MaterialApp(
  onGenerateRoute: (settings) {
    if (settings.name == '/product') {
      final args = settings.arguments as Map<String, dynamic>;
      return MaterialPageRoute(
        builder: (context) {
          return ProductScreen(
            productId: args['id'],
            productName: args['name'],
          );
        },
      );
    }
    // Handle other routes or return a "not found" page
    return MaterialPageRoute(builder: (context) => NotFoundScreen());
  },
)
```

This approach gives you complete control over how routes are generated, allowing for dynamic behavior based on the route name and arguments.

## Navigator 2.0: The Declarative Approach

While Navigator 1.0 works well for many apps, it has limitations. It's harder to handle deep linking (opening specific screens from outside your app), web navigation with browser history, and complex navigation patterns. Enter Navigator 2.0, also known as the Router API.

Navigator 2.0 takes a declarative approach: instead of imperatively telling Flutter to "push this screen" or "pop that screen," you declare the entire navigation stack based on your app's current state.

Here's a high-level overview of the components:

1. **Router**: Manages the overall routing strategy
2. **RouteInformationParser**: Converts route information (like URLs) into your app's navigation state
3. **RouterDelegate**: Builds the Navigator with the appropriate pages based on the current state

Let's be honest—Navigator 2.0 has a steep learning curve and quite a bit of boilerplate code. That's why most developers use packages that simplify the experience. The most popular as of 2025 is `go_router`, which gives you the power of Navigator 2.0 without the complexity.

### Simplified Navigator 2.0 with go_router

Let's see how `go_router` makes declarative navigation more approachable:

```dart
// First, define your router
final _router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => HomeScreen(),
      routes: [
        GoRoute(
          path: 'details/:id',
          builder: (context, state) {
            final id = state.params['id']!;
            return DetailsScreen(id: id);
          },
        ),
        GoRoute(
          path: 'settings',
          builder: (context, state) => SettingsScreen(),
        ),
      ],
    ),
  ],
);

// Then use it in your MaterialApp
MaterialApp.router(
  routerConfig: _router,
  // Other MaterialApp properties
)

// To navigate
ElevatedButton(
  child: Text('View Details'),
  onPressed: () {
    context.go('/details/123');  // Using GoRouter extension
  },
)

// Or with parameters
context.go('/details/${product.id}');
```

This approach combines the organization of named routes with the power of the Router API. Your routes are clearly defined, path parameters are easy to work with, and you get deep linking support out of the box.

### Handling Authentication with Navigation

A common challenge in app navigation is handling authenticated routes—screens that should only be accessible to logged-in users. With `go_router`, this is straightforward:

```dart
final _router = GoRouter(
  redirect: (context, state) {
    // Check if the user is logged in
    final isLoggedIn = AuthService.isLoggedIn;
    
    // If trying to access authenticated routes while not logged in,
    // redirect to login
    final isGoingToAuthenticatedRoute = state.location.startsWith('/dashboard');
    if (isGoingToAuthenticatedRoute && !isLoggedIn) {
      return '/login';
    }
    
    // If already logged in and going to login, redirect to dashboard
    if (isLoggedIn && state.location == '/login') {
      return '/dashboard';
    }
    
    // No redirect needed
    return null;
  },
  routes: [
    GoRoute(
      path: '/login',
      builder: (context, state) => LoginScreen(),
    ),
    GoRoute(
      path: '/dashboard',
      builder: (context, state) => DashboardScreen(),
    ),
    // Other routes
  ],
)
```

The `redirect` function is called whenever navigation happens, giving you a chance to intercept and redirect if needed.

## Advanced Navigation Patterns

Now that we've covered the basics of both Navigator 1.0 and the Router API, let's explore some advanced navigation patterns that you'll encounter in modern apps.

### Bottom Tab Navigation

Tab navigation is ubiquitous in mobile apps—think of the bottom tabs in Instagram or TikTok. Flutter makes this easy with `BottomNavigationBar`:

```dart
class HomeScreen extends StatefulWidget {
  @override
  _HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  int _selectedIndex = 0;
  
  final List<Widget> _screens = [
    FeedScreen(),
    SearchScreen(),
    ProfileScreen(),
  ];
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: _screens[_selectedIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _selectedIndex,
        onTap: (index) {
          setState(() {
            _selectedIndex = index;
          });
        },
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Feed'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}
```

This approach works well for simple tab navigation, but it doesn't preserve state when switching between tabs. If you want to maintain the scroll position or other state in each tab, you need to use `IndexedStack` instead of directly swapping widgets:

```dart
// Replace the body with
body: IndexedStack(
  index: _selectedIndex,
  children: _screens,
),
```

With `IndexedStack`, all screens are built and maintained in memory, just hidden when not selected.

### Drawer Navigation

Another common pattern is drawer navigation, where a side menu slides in from the edge of the screen. Flutter's `Drawer` widget makes this easy:

```dart
Scaffold(
  appBar: AppBar(title: Text('My App')),
  drawer: Drawer(
    child: ListView(
      padding: EdgeInsets.zero,
      children: [
        DrawerHeader(
          decoration: BoxDecoration(color: Colors.blue),
          child: Text('Menu', style: TextStyle(color: Colors.white, fontSize: 24)),
        ),
        ListTile(
          leading: Icon(Icons.home),
          title: Text('Home'),
          onTap: () {
            Navigator.pop(context); // Close the drawer
            Navigator.pushNamed(context, '/');
          },
        ),
        ListTile(
          leading: Icon(Icons.settings),
          title: Text('Settings'),
          onTap: () {
            Navigator.pop(context); // Close the drawer
            Navigator.pushNamed(context, '/settings');
          },
        ),
        // More menu items
      ],
    ),
  ),
  body: // Your screen content
)
```

The drawer automatically connects to the `Scaffold` and includes the "hamburger menu" icon in the app bar.

### Nested Navigation

Sometimes you need complex navigation hierarchies, like tabs that each have their own navigation stack. This is where nested navigation comes in:

```dart
class MainScreen extends StatefulWidget {
  @override
  _MainScreenState createState() => _MainScreenState();
}

class _MainScreenState extends State<MainScreen> {
  int _selectedIndex = 0;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: IndexedStack(
        index: _selectedIndex,
        children: [
          Navigator(
            onGenerateRoute: (settings) {
              return MaterialPageRoute(
                builder: (context) => FeedScreen(),
              );
            },
          ),
          Navigator(
            onGenerateRoute: (settings) {
              return MaterialPageRoute(
                builder: (context) => SearchScreen(),
              );
            },
          ),
          Navigator(
            onGenerateRoute: (settings) {
              return MaterialPageRoute(
                builder: (context) => ProfileScreen(),
              );
            },
          ),
        ],
      ),
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _selectedIndex,
        onTap: (index) {
          setState(() {
            _selectedIndex = index;
          });
        },
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Feed'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }
}
```

Each tab now has its own `Navigator`, allowing for independent navigation stacks within each tab.

With `go_router`, nested navigation becomes even more elegant:

```dart
final _router = GoRouter(
  routes: [
    ShellRoute(
      builder: (context, state, child) {
        return ScaffoldWithBottomNavBar(child: child);
      },
      routes: [
        GoRoute(
          path: '/feed',
          builder: (context, state) => FeedScreen(),
          routes: [
            GoRoute(
              path: 'details/:id',
              builder: (context, state) => FeedDetailScreen(
                id: state.params['id']!,
              ),
            ),
          ],
        ),
        GoRoute(
          path: '/search',
          builder: (context, state) => SearchScreen(),
        ),
        GoRoute(
          path: '/profile',
          builder: (context, state) => ProfileScreen(),
        ),
      ],
    ),
  ],
)
```

The `ShellRoute` maintains the bottom navigation bar across different routes, while each tab route can have its own nested routes.

## Deep Linking: Connecting Your App to the World

Deep linking allows users to navigate directly to specific content in your app from external sources, like links in emails or other apps. It's a crucial feature for modern apps, and Flutter's Router API makes it relatively straightforward.

For Android, you need to update your `AndroidManifest.xml` to define the deep link scheme:

```xml
<activity
    android:name=".MainActivity"
    android:launchMode="singleTop"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" android:host="open" />
    </intent-filter>
</activity>
```

For iOS, you need to update your `Info.plist`:

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>CFBundleURLName</key>
        <string>com.example.myapp</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>myapp</string>
        </array>
    </dict>
</array>
```

With these configurations, your app can handle links like `myapp://open/product/123`. Using `go_router`, processing these links is automatic—the router matches the path to your defined routes and navigates accordingly.

## Real-World Example: Building a Social Media App Navigator

Let's bring everything together with a real-world example: building the navigation structure for a social media app. We'll include a feed, search, notifications, and profile tabs, each with their own navigation stack, plus a direct messaging feature accessible from multiple places.

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

// Screens
import 'screens/feed_screen.dart';
import 'screens/search_screen.dart';
import 'screens/notifications_screen.dart';
import 'screens/profile_screen.dart';
import 'screens/post_detail_screen.dart';
import 'screens/user_profile_screen.dart';
import 'screens/direct_messages_screen.dart';
import 'screens/conversation_screen.dart';
import 'screens/settings_screen.dart';
import 'screens/login_screen.dart';
import 'screens/signup_screen.dart';

// Services
import 'services/auth_service.dart';

class SocialApp extends StatelessWidget {
  SocialApp({Key? key}) : super(key: key);

  // Auth service to check login status
  final authService = AuthService();

  // Navigation logic
  late final _router = GoRouter(
    // Redirect based on authentication status
    redirect: (context, state) {
      final isLoggedIn = authService.isLoggedIn;
      final isLoginRoute = state.location == '/login' || 
                           state.location == '/signup';
      
      // If not logged in and not on login route, redirect to login
      if (!isLoggedIn && !isLoginRoute) {
        return '/login';
      }
      
      // If logged in and on login route, redirect to feed
      if (isLoggedIn && isLoginRoute) {
        return '/feed';
      }
      
      // No redirect needed
      return null;
    },
    
    // Route refresh listener
    refreshListenable: authService,
    
    // Route definitions
    routes: [
      // Authentication routes
      GoRoute(
        path: '/login',
        builder: (context, state) => LoginScreen(),
      ),
      GoRoute(
        path: '/signup',
        builder: (context, state) => SignupScreen(),
      ),
      
      // Main app shell with bottom navigation
      ShellRoute(
        builder: (context, state, child) {
          return MainScreenWithBottomNav(child: child);
        },
        routes: [
          // Feed tab and its nested routes
          GoRoute(
            path: '/feed',
            builder: (context, state) => FeedScreen(),
            routes: [
              GoRoute(
                path: 'post/:postId',
                builder: (context, state) {
                  return PostDetailScreen(
                    postId: state.params['postId']!,
                  );
                },
              ),
            ],
          ),
          
          // Search tab and its nested routes
          GoRoute(
            path: '/search',
            builder: (context, state) => SearchScreen(),
          ),
          
          // Notifications tab and its nested routes
          GoRoute(
            path: '/notifications',
            builder: (context, state) => NotificationsScreen(),
          ),
          
          // Profile tab and its nested routes
          GoRoute(
            path: '/profile',
            builder: (context, state) => ProfileScreen(),
            routes: [
              GoRoute(
                path: 'settings',
                builder: (context, state) => SettingsScreen(),
              ),
            ],
          ),
        ],
      ),
      
      // Direct messaging routes (accessible from multiple places)
      GoRoute(
        path: '/messages',
        builder: (context, state) => DirectMessagesScreen(),
        routes: [
          GoRoute(
            path: ':userId',
            builder: (context, state) {
              return ConversationScreen(
                userId: state.params['userId']!,
              );
            },
          ),
        ],
      ),
      
      // User profile route (accessible from multiple places)
      GoRoute(
        path: '/user/:userId',
        builder: (context, state) {
          return UserProfileScreen(
            userId: state.params['userId']!,
          );
        },
      ),
    ],
    
    // Error handling
    errorBuilder: (context, state) => ErrorScreen(
      error: state.error.toString(),
    ),
  );
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Social App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      routerConfig: _router,
    );
  }
}

// Bottom navigation implementation
class MainScreenWithBottomNav extends StatefulWidget {
  final Widget child;
  
  const MainScreenWithBottomNav({Key? key, required this.child}) : super(key: key);
  
  @override
  _MainScreenWithBottomNavState createState() => _MainScreenWithBottomNavState();
}

class _MainScreenWithBottomNavState extends State<MainScreenWithBottomNav> {
  int _calculateSelectedIndex(BuildContext context) {
    final location = GoRouterState.of(context).location;
    if (location.startsWith('/feed')) {
      return 0;
    }
    if (location.startsWith('/search')) {
      return 1;
    }
    if (location.startsWith('/notifications')) {
      return 2;
    }
    if (location.startsWith('/profile')) {
      return 3;
    }
    return 0;
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: widget.child,
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _calculateSelectedIndex(context),
        onTap: (index) {
          switch (index) {
            case 0:
              context.go('/feed');
              break;
            case 1:
              context.go('/search');
              break;
            case 2:
              context.go('/notifications');
              break;
            case 3:
              context.go('/profile');
              break;
          }
        },
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Feed'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
          BottomNavigationBarItem(icon: Icon(Icons.notifications), label: 'Notifications'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
        type: BottomNavigationBarType.fixed,
      ),
    );
  }
}
```

This example demonstrates:

- Authentication-based redirects
- A bottom navigation bar with tabs
- Nested navigation within tabs
- Routes accessible from multiple places
- Deep linking support (via go_router's path matching)
- Error handling for invalid routes

With this structure, users can navigate between tabs, dive deeper into specific content, and directly access content via deep links—all with a consistent, maintainable codebase.

## Navigation Best Practices

Before we wrap up, let's review some best practices for Flutter navigation:

1. **Choose the right approach for your app's complexity**:
    - For simple apps, Navigator 1.0 is often sufficient
    - For complex navigation with deep linking, use the Router API or a package like go_router
2. **Plan your navigation structure early**:
    - Map out your screens and how they connect
    - Consider creating a visual diagram of your navigation flows
3. **Be consistent with transitions**:
    - Use platform-appropriate transitions (e.g., Material for Android, Cupertino for iOS)
    - Maintain consistent transition patterns throughout your app
4. **Handle back navigation gracefully**:
    - Confirm before exiting if the user might lose data
    - Consider the Android back button behavior
5. **Optimize for performance**:
    - Don't keep too many screens in memory
    - Use lazy loading for content when appropriate
6. **Test navigation thoroughly**:
    - Verify deep links work as expected
    - Ensure navigation history behaves correctly after background/foreground transitions
7. **Use meaningful route names**:
    - Follow REST-like conventions for named routes
    - Consider using path parameters for dynamic content
8. **Handle errors**:
    - Provide a graceful 404 experience for invalid routes
    - Catch and report navigation errors

## Conclusion

Navigation is the invisible architecture that shapes how users experience your app. By mastering Flutter's navigation systems, you can create apps that feel intuitive, responsive, and professional.

We've covered a lot in this chapter—from the basics of push/pop navigation to complex patterns with nested navigators and deep linking. The key is to start simple and add complexity only as your app requires it. For most projects, go_router offers an excellent balance of power and simplicity, bridging the gap between Navigator 1.0's ease of use and Navigator 2.0's capabilities.

In the next chapter, we'll explore Flutter's theming and styling capabilities, learning how to create visually cohesive apps that stand out with your unique brand identity. We'll look at Material Design 3, custom theming, responsive design, and animations that can take your navigation experiences to the next level.

Before we move on, I'd love to hear your thoughts on navigation. Have you implemented complex navigation patterns in your Flutter apps? Are there specific navigation challenges you've encountered? Your input will help shape our upcoming chapters to address real-world Flutter development needs.

<div style="text-align: center">⁂</div>

[^1]: https://docs.flutter.dev/cookbook/navigation/navigation-basics

[^2]: https://www.reddit.com/r/FlutterDev/comments/u43t86/routingnavigation_in_flutter/

[^3]: https://blog.codemagic.io/flutter-navigator2/

[^4]: https://www.reddit.com/r/flutterhelp/comments/1fbszv3/best_practices_for_routingnavigation_within/

[^5]: https://www.youtube.com/watch?v=_13UW57UmQA

[^6]: https://dev.to/yatendra2001/mastering-navigation-in-flutter-a-comprehensive-guide-1p1l

[^7]: https://www.taylorfrancis.com/books/9781003104636/chapters/10.1201/9781003104636-14

[^8]: https://docs.flutter.dev/ui/navigation

[^9]: https://www.dhiwise.com/post/deep-dive-into-flutter-routing-everything-you-need-to-know

[^10]: https://www.reddit.com/r/FlutterDev/comments/k9abo2/navigator_20_is_very_complicated/

[^11]: https://www.vogella.com/tutorials/FlutterNavigation/article.html

[^12]: https://docs.flutter.dev/resources/books

[^13]: https://www.youtube.com/watch?v=DlArCl8jvlo

[^14]: https://www.kodeco.com/books/flutter-apprentice/v3.0/chapters/7-routes-navigation

[^15]: https://www.barnesandnoble.com/w/beginning-flutter-marco-l-napoli/1130003443

[^16]: https://docs.flutter.dev/cookbook/navigation/named-routes

[^17]: https://www.freecodecamp.org/news/navigation-in-flutter/

[^18]: https://www.youtube.com/watch?v=C6nTXjQFVKI

[^19]: https://stackoverflow.com/questions/68358147/which-is-the-best-way-of-routing-in-a-flutter-app

[^20]: https://blog.openreplay.com/deep-linking-in-flutter/

[^21]: https://stackoverflow.com/questions/67838448/flutter-navigator-2-0-named-routes-with-drawer-navigation

[^22]: https://www.linkedin.com/pulse/deep-dive-flutter-routing-everything-you-need-know-crifly-arjce

[^23]: https://www.scaler.com/topics/deeplink-flutter/

[^24]: https://docs.flutter.dev/cookbook/navigation/navigate-with-arguments

[^25]: https://www.youtube.com/watch?v=g8xGxd99Fgs

[^26]: https://pieces.app/blog/flutter-navigation-overview-and-best-practices

[^27]: https://docs.flutter.dev/ui/navigation/deep-linking

[^28]: https://www.wiley.com/en-us/Flutter+For+Dummies-p-00039002

[^29]: https://www.walmart.com/ip/Flutter-in-7-Days-Build-user-friendly-apps-with-widgets-and-navigation-English-Edition-Paperback-9789355516091/7246551397

[^30]: https://docs.flutter.dev/cookbook/navigation

