# 🎯 **Feature-based Modular Clean Architecture in Flutter**

Welcome to the ultimate guide for **Feature-based Modular Clean Architecture** using **Riverpod** in Flutter! 🚀 This repository demonstrates how to build **scalable**, **testable**, and **maintainable** applications using **Uncle Bob’s Clean Architecture principles** and a **feature-based modular structure**.

---

## 🧠 **What is Clean Architecture?**

### Key Principles:
1. **Separation of Concerns**: Each layer of the application has a specific responsibility. No layer does what another should do. 🧩
2. **Dependency Rule**: Inner layers (like the domain layer) don’t depend on outer layers (like the data or presentation layer). Instead, dependencies are inverted using **abstractions**. 🔄
3. **Testability**: Clear boundaries and dependency inversion make unit testing easy and effective. ✅

---
### 🔍 **Layer Breakdown**

Clean Architecture divides your app into three **core layers**, each with a specific purpose. This structure ensures **scalability**, **testability**, and **maintainability**. Let’s dive deeper into each layer! 🌊

---

### 1️⃣ **Domain Layer** (Inner Circle 🏛️)
- **Purpose**: The **heart** of the application. Contains **business rules** that define how data is manipulated and used, independent of external frameworks.
- **Characteristics**:
  - Completely **framework-agnostic** (doesn't depend on Flutter or any library).
  - Focuses only on the **"what"** and **"why"** of the app logic, not the **"how"**.
- **Components**:
  - **Entities**:
    - Represent the core objects of the app (e.g., `UserEntity`, `StudentEntity`).
    - Pure data objects without behavior or dependencies (just fields and constructors).
    - Example:
      ```dart
      class UserEntity {
        final String id;
        final String name;
        final String email;

        UserEntity({required this.id, required this.name, required this.email});
      }
      ```
  - **Abstract Repositories**:
    - Define contracts or interfaces for data handling, like `getUser()` or `login()`.
    - These interfaces **decouple** the domain layer from data layer implementations.
    - Example:
      ```dart
      abstract class AuthRepository {
        Future<UserEntity> login(String email, String password);
        Future<void> logout();
      }
      ```
  - **Use Cases**:
    - Encapsulate **application-specific business logic** into single-responsibility classes.
    - Each use case performs a specific action (e.g., `LoginUseCase`, `FetchProfileUseCase`).
    - These are invoked by the **presentation layer**.
    - Example:
      ```dart
      class LoginUseCase {
        final AuthRepository repository;

        LoginUseCase(this.repository);

        Future<UserEntity> call(String email, String password) {
          return repository.login(email, password);
        }
      }
      ```

---

### 2️⃣ **Data Layer** (Outer Circle 🗄️)
- **Purpose**: Acts as a **bridge** between the app and external data sources like APIs or local databases. Transforms raw data into domain objects (entities).
- **Characteristics**:
  - Contains all **framework-dependent code** (like HTTP clients, database libraries).
  - Implements the contracts defined in the **domain layer's repositories**.
- **Components**:
  - **Models**:
    - Represent data in a format used by APIs or databases (e.g., `UserModel`, `StudentModel`).
    - Include serialization/deserialization logic for JSON or database mapping.
    - Example:
      ```dart
      class UserModel {
        final String id;
        final String name;
        final String email;

        UserModel({required this.id, required this.name, required this.email});

        factory UserModel.fromJson(Map<String, dynamic> json) {
          return UserModel(
            id: json['id'],
            name: json['name'],
            email: json['email'],
          );
        }
      }
      ```
  - **Data Sources**:
    - Responsible for **fetching data** from external sources.
    - Split into:
      - **Remote Data Sources**: Fetch data from APIs or cloud (e.g., Firebase).
      - **Local Data Sources**: Fetch data from local storage (e.g., SQLite, SharedPreferences).
    - Example:
      ```dart
      class AuthRemoteDataSource {
        final Dio dio;

        AuthRemoteDataSource(this.dio);

        Future<Map<String, dynamic>> login(String email, String password) async {
          final response = await dio.post('/login', data: {'email': email, 'password': password});
          return response.data;
        }
      }
      ```
  - **Repository Implementations**:
    - Concrete implementations of **domain layer's repositories**.
    - Aggregate data from multiple sources (remote/local) and return domain objects.
    - Example:
      ```dart
      class AuthRepositoryImpl implements AuthRepository {
        final AuthRemoteDataSource remoteDataSource;

        AuthRepositoryImpl(this.remoteDataSource);

        @override
        Future<UserEntity> login(String email, String password) async {
          final data = await remoteDataSource.login(email, password);
          return UserEntity(id: data['id'], name: data['name'], email: data['email']);
        }
      }
      ```

---

### 3️⃣ **Presentation Layer** (Outer Circle 🎨)
- **Purpose**: Manages **UI components** and **user interactions**. It’s the **face** of the app.
- **Characteristics**:
  - Relies heavily on **state management** tools (e.g., Riverpod) to communicate with the domain layer.
  - Contains **Flutter-dependent code** like `Widgets`, `Pages`, and `Providers`.
- **Components**:
  - **Pages**:
    - Complete screens (e.g., `LoginPage`, `HomePage`).
    - Use **providers** or state management to fetch and display data.
    - Example:
      ```dart
      class LoginPage extends ConsumerWidget {
        @override
        Widget build(BuildContext context, WidgetRef ref) {
          final authState = ref.watch(authProvider);

          return Scaffold(
            body: authState.isLoading
                ? CircularProgressIndicator()
                : LoginForm(onLogin: (email, password) {
                    ref.read(authProvider.notifier).login(email, password);
                  }),
          );
        }
      }
      ```
  - **Widgets**:
    - Reusable UI components (e.g., `LoginForm`, `ProfileCard`).
    - Designed to be **feature-specific** or **shared** across features.
    - Example:
      ```dart
      class LoginForm extends StatelessWidget {
        final Function(String, String) onLogin;

        LoginForm({required this.onLogin});

        @override
        Widget build(BuildContext context) {
          return Column(
            children: [
              TextField(decoration: InputDecoration(labelText: 'Email')),
              TextField(decoration: InputDecoration(labelText: 'Password'), obscureText: true),
              ElevatedButton(onPressed: () => onLogin('test@example.com', 'password123'), child: Text('Login'))
            ],
          );
        }
      }
      ```
  - **State Management**:
    - **Riverpod** is used to manage state and connect the UI with use cases.
    - Providers ensure reactive updates to UI when state changes.
    - Example:
      ```dart
      final authProvider = StateNotifierProvider<AuthNotifier, AuthState>(
        (ref) => AuthNotifier(authRepository: ref.read(authRepositoryProvider)),
      );
      ```
---

## 🌟 **Feature-based Modular Structure**

Instead of grouping files by type (e.g., all models in one folder), we **modularize** the app by features. Each **feature** is **self-contained** and includes its own:
- **Data** layer
- **Domain** layer
- **Presentation** layer

For example:
```plaintext
features/
  └── authentication/
      ├── data/
      ├── domain/
      └── presentation/
```

### Why modular? 🤔
- **Scalability**: New features can be added without affecting the rest of the app.
- **Maintainability**: Isolates features for easier debugging and enhancements.
- **Team Collaboration**: Teams can work on separate modules independently.

---

## 📂 **Generalized Directory Structure**

Here's a **complete, modular directory structure** for a **sample app**:

```plaintext
your_next_awesome_app/
│
├── android/                # Android platform-specific files
├── ios/                    # iOS platform-specific files
├── web/                    # Web platform-specific files
├── test/                   # Unit and widget tests
│
└── lib/                    # Main Flutter application
    │
    ├── main.dart           # Entry point of the app
    │
    ├── core/               # App-wide reusable code
    │   ├── config/         # Configuration settings
    │   │   ├── app_config.dart       # App configuration settings
    │   │   └── environment_config.dart # Environment-specific configs
    │   │
    │   ├── constants/      # App constants
    │   │   ├── app_constants.dart    # General constants
    │   │   ├── api_constants.dart    # API-related constants
    │   │   └── route_constants.dart  # Navigation-related constants
    │   │
    │   ├── error/          # Error handling utilities
    │   │   ├── exceptions.dart       # App-specific exceptions
    │   │   └── failures.dart         # Failure models
    │   │
    │   ├── network/        # Networking utilities
    │   │   ├── network_info.dart     # Connectivity status checker
    │   │   └── http_client.dart      # HTTP client setup (e.g., Dio)
    │   │
    │   ├── routes/         # Navigation management
    │   │   ├── app_router.dart       # Route definitions
    │   │   └── route_observer.dart   # Route observer for analytics
    │   │
    │   ├── services/       # Global app services
    │   │   ├── local_storage_service.dart # Local storage utilities
    │   │   ├── secure_storage_service.dart # Secure storage utilities
    │   │   └── notification_service.dart  # Notification utilities
    │   │
    │   └── theme/          # App-wide themes and styles
    │       ├── app_theme.dart        # Main theme configuration
    │       ├── color_scheme.dart     # App color scheme
    │       ├── text_styles.dart      # Text styles
    │       └── theme_provider.dart   # Theme management
    │
    ├── features/           # Feature-based modules
    │   ├── feature_one/    # Feature 1
    │   │   ├── data/       # Data layer
    │   │   │   ├── datasources/
    │   │   │   │   ├── local_datasource.dart
    │   │   │   │   └── remote_datasource.dart
    │   │   │   ├── models/
    │   │   │   │   └── feature_one_model.dart
    │   │   │   └── repositories/
    │   │   │       └── feature_one_repository_impl.dart
    │   │   │
    │   │   ├── domain/     # Domain layer
    │   │   │   ├── entities/
    │   │   │   │   └── feature_one_entity.dart
    │   │   │   ├── repositories/
    │   │   │   │   └── feature_one_repository.dart
    │   │   │   └── usecases/
    │   │   │       ├── get_feature_one_data.dart
    │   │   │       └── update_feature_one_data.dart
    │   │   │
    │   │   └── presentation/ # Presentation layer
    │   │       ├── pages/
    │   │       │   ├── feature_one_page.dart
    │   │       │   └── feature_one_details_page.dart
    │   │       ├── providers/
    │   │       │   └── feature_one_provider.dart
    │   │       └── widgets/
    │   │           ├── feature_one_widget.dart
    │   │           └── feature_one_card.dart
    │   │
    │   └── feature_two/    # Feature 2 (similar structure as feature_one)
    │
    ├── shared/             # Shared components across features
    │   ├── widgets/        # Reusable widgets
    │   │   ├── custom_button.dart
    │   │   ├── loading_indicator.dart
    │   │   ├── error_display.dart
    │   │   └── empty_state_widget.dart
    │   │
    │   ├── models/         # Shared data models
    │   │   └── pagination_model.dart
    │   │
    │   └── utils/          # Utility functions
    │       ├── date_formatter.dart
    │       ├── input_validator.dart
    │       └── debouncer.dart
    │
    └── l10n/               # Localization files for internationalization
        ├── app_en.arb      # English translations
        ├── app_es.arb      # Spanish translations
        └── app_fr.arb      # French translations

```
Here’s an expanded directory structure with additional details for each folder and suggested Dart files within them. Each file is named according to its purpose to help maintain clarity and organization:

---

## **Directory Breakdown**

### **`android/`, `ios/`, `web/`**
- **Purpose**: Platform-specific code generated by Flutter. Typically, you won’t need to modify these unless you're integrating native functionalities (e.g., camera, sensors, or deep linking).

---

### **`test/`**
- **Purpose**: Contains unit tests, widget tests, and integration tests for the app.

---

### **`lib/`**

#### 1️⃣ **`main.dart`**
- **Purpose**: Entry point of the Flutter app.
- Contains:
  - `runApp()`: Initialize the app.
  - `MaterialApp` or `GoRouter` configuration.
  - Dependency injection initialization.

---

#### 2️⃣ **`core/`**
**Reusable, app-wide utilities and configurations.**

##### **`config/`**
- Stores app-wide configuration settings.
- **Files**:
  - `environment_config.dart`: Handles environment variables (e.g., dev/prod URLs).
  - `app_config.dart`: Stores general app configuration, like version or app name.

##### **`constants/`**
- Central location for static constants.
- **Files**:
  - `app_colors.dart`: Defines color palette for the app.
  - `app_fonts.dart`: Specifies text styles and fonts.
  - `app_strings.dart`: Contains static strings used across the app.

##### **`error/`**
- Handles app-wide error management.
- **Files**:
  - `failure.dart`: Defines common error classes (e.g., `NetworkFailure`).
  - `error_handler.dart`: Custom error handling logic for the app.

##### **`network/`**
- Networking utilities.
- **Files**:
  - `dio_client.dart`: Configures Dio for API calls.
  - `api_constants.dart`: Defines base URLs and endpoints.

##### **`routes/`**
- Centralizes navigation logic.
- **Files**:
  - `app_routes.dart`: Defines all app routes.
  - `route_names.dart`: Contains string constants for route names.

##### **`services/`**
- App services for non-UI logic.
- **Files**:
  - `notification_service.dart`: Push notification handling (e.g., Firebase Messaging).
  - `analytics_service.dart`: Logs analytics events (e.g., Google Analytics).

##### **`theme/`**
- App-wide styles and themes.
- **Files**:
  - `app_theme.dart`: Defines light and dark themes.
  - `theme_constants.dart`: Shared styling constants (e.g., padding values).

---

#### 3️⃣ **`features/`**
**Feature-based modularization for better scalability.**

##### **`feature_one/`**
Handles login, registration, and logout functionalities.
- **Folders and Files**:
  - `data/`: Data handling.
    - `auth_remote_data_source.dart`: API calls for login/register.
    - `auth_repository_impl.dart`: Implements domain repository.
  - `domain/`: Business logic.
    - `auth_entity.dart`: Defines `UserEntity`.
    - `auth_repository.dart`: Abstract repository for auth logic.
    - `login_usecase.dart`: Encapsulates login operation.
  - `presentation/`: UI components.
    - `login_page.dart`: Login screen.
    - `login_form.dart`: Widget for the login form.
  - `state/`:
    - `auth_provider.dart`: Riverpod state provider for authentication.

##### **`student_profile/`**
Manages user profile functionalities.
- **Folders and Files**:
  - `data/`:
    - `profile_remote_data_source.dart`: Fetch/update profile data from the API.
    - `profile_repository_impl.dart`: Concrete implementation.
  - `domain/`:
    - `student_entity.dart`: Core entity for student profile.
    - `profile_repository.dart`: Abstract repository.
    - `fetch_profile_usecase.dart`: Fetches student profile details.
  - `presentation/`:
    - `profile_page.dart`: Displays the user’s profile.
    - `profile_card.dart`: Widget for showing user info.

##### **`timetable/`**
Handles timetable management.
- **Folders and Files**:
  - `data/`:
    - `timetable_remote_data_source.dart`: API calls to fetch timetable.
  - `domain/`:
    - `timetable_entity.dart`: Core entity for timetable.
  - `presentation/`:
    - `timetable_page.dart`: UI for timetable display.
    - `day_timetable_card.dart`: Widget for daily schedule.

##### **`attendance/`**
Handles attendance tracking and visualization.
- **Folders and Files**:
  - `data/`:
    - `attendance_remote_data_source.dart`: API calls for attendance data.
  - `domain/`:
    - `attendance_entity.dart`: Core entity for attendance.
  - `fetch_attendance_usecase.dart`: Fetches attendance details.
  - `presentation/`:
    - `attendance_page.dart`: UI for showing attendance summary.

---

#### 4️⃣ **`shared/`**
**Code shared across multiple features.**

##### **`widgets/`**
- Reusable UI components.
- **Files**:
  - `custom_button.dart`: General-purpose button widget.
  - `loading_indicator.dart`: Standard loading spinner.
  - `error_widget.dart`: Widget to display errors.

##### **`models/`**
- Common models used in the app.
- **Files**:
  - `base_response.dart`: Generic model for API responses.
  - `pagination.dart`: Handles paginated data.

##### **`utils/`**
- General utility functions.
- **Files**:
  - `date_formatter.dart`: Functions for date formatting.
  - `validators.dart`: Input validation logic (e.g., email validation).

---

#### 5️⃣ **`l10n/`**
- Localization files for internationalization.
- **Files**:
  - `app_en.arb`: English translations.
  - `app_hi.arb`: Hindi translations.
  - `app_fr.arb`: French translations.

---

### 🌟 **Final Takeaways**
This modular structure ensures your app is:
- **Scalable**: Easy to add features without breaking existing ones.
- **Testable**: Each feature and layer can be independently tested.
- **Maintainable**: Clear separation of concerns simplifies debugging and updates.

---

## 🛠️ **Recommended Libraries with Alternatives**
1. **State Management**
   - **Recommended**: **Riverpod** 🏞️ (Simple, modern, and robust for dependency injection and state management.)
   - **Alternatives**:
     - **BLoC (Business Logic Component)**: Structured and event-driven.
     - **Provider**: Lightweight and widely adopted.
     - **Redux**: Centralized state management with a unidirectional data flow.

2. **HTTP Client**
   - **Recommended**: **Dio** 🌐 (Powerful HTTP client with built-in features like interceptors, cancellation, and file uploading.)
   - **Alternatives**:
     - **HTTP (Flutter's HTTP library)**: Lightweight and simple.
     - **Chopper**: HTTP client with Retrofit-style generation.
     - **GraphQL Flutter**: For GraphQL-based APIs.

3. **Routing**
   - **Recommended**: **GoRouter** 🗺️ (Declarative routing with deep link support and navigation guards.)
   - **Alternatives**:
     - **AutoRoute**: Code generation-based routing with deep linking support.
     - **Flutter Navigator 2.0**: Built-in Flutter navigation, although more manual.

4. **Dependency Injection**
   - **Recommended**: **Riverpod** 💉 (Integrated DI and state management.)
   - **Alternatives**:
     - **GetIt**: A service locator approach for DI.
     - **Injectable**: Works on top of GetIt with code generation.
     - **Flutter BLoC**: Includes built-in dependency injection capabilities.

5. **Internationalization (i18n)**
   - **Recommended**: **Flutter Intl** 🌍 (Easy ARB-based i18n management with IDE integrations.)
   - **Alternatives**:
     - **Easy Localization**: Simplifies translation management and supports JSON, YAML, and CSV.
     - **intl (Official Dart Library)**: Core library for date/number formatting and i18n handling.

6. **Database/Local Storage**
   - **Recommended**: 
     - **Hive** 🐝 (Fast, key-value database for Flutter apps.)
     - **Drift (formerly Moor)** 📜 (Database abstraction layer with SQLite.)
   - **Alternatives**:
     - **SharedPreferences**: Basic key-value storage for simple use cases.
     - **Isar**: A new Flutter-native database optimized for speed.
     - **ObjectBox**: High-performance NoSQL database.

7. **Testing**
   - **Recommended**:
     - **Flutter Test**: Official testing framework.
     - **Mockito**: Mocking library for Dart/Flutter tests.
   - **Alternatives**:
     - **Mocktail**: A modern alternative to Mockito.
     - **Bloc Test**: For BLoC-specific tests.

8. **Analytics/Crash Reporting**
   - **Recommended**:
     - **Firebase Analytics/Crashlytics** 📊
   - **Alternatives**:
     - **Sentry**: Advanced crash reporting and monitoring.
     - **AppCenter**: An all-in-one solution for CI/CD and crash analytics.

9. **UI Toolkit**
   - **Recommended**:
     - **Material Design** (Flutter’s default).
     - **Flutter Neumorphic**: For unique, neumorphic designs.
   - **Alternatives**:
     - **GetWidget**: A collection of pre-built widgets.
     - **VelocityX**: A minimalist UI toolkit.

---

### Why Alternatives Matter:
Providing alternatives empowers developers to choose tools that best suit their preferences, project scale, or team expertise. These options ensure flexibility and scalability in any Flutter project. 🚀

---

## 📝 **How to Use this Repository**

1. **Clone the repo**:
   ```bash
   git clone https://github.com/your_username/student_app_clean_architecture.git
   cd student_app_clean_architecture
   ```
2. **Run the app**:
   ```bash
   flutter pub get
   flutter run
   ```
3. **Explore by Features**:
   - Check the `features/` folder for feature-specific implementations.
   - Dive into `core/` for reusable utilities.

---

## 📚 **Learning Resources**

- [Clean Architecture by Uncle Bob](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Flutter Riverpod Documentation](https://riverpod.dev)
- [Flutter Intl for Localization](https://pub.dev/packages/flutter_intl)

---

## 🤝 **Contributing**

Feel free to contribute to this repository! 💡
1. Fork the repo.
2. Create a new branch.
3. Make changes and test them.
4. Submit a pull request. 🚀

---

## 🛡️ **License**

This project is licensed under the MIT License. 📝

---
