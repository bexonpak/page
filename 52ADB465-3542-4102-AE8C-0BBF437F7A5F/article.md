iOS app 的 Clean Architecture + MVVM 架構的一個 Demo

#### 螢幕截圖

<img width="1707" alt="user_list_app_9_21" src="user_list_app_9_21.png">

#### 文件架構

```
.
├── Feature/
│   ├── Data/ 
│   │   ├── Network/
│   │   │   ├── CloudUserApi.swift
│   │   │   ├── CloudUserRequest.swift
│   │   │   └── CloudUserResponse.swift
│   │   └── Service/
│   │       └── CloudApiService.swift
│   ├── Domain/ 
│   │   ├── Entities/
│   │   │   └── UserEntity.swift
│   │   └── UseCases/
│   │       ├── CloudUserUseCase.swift
│   │       └── CloudSyncUseCase.swift
│   ├── Presentation/ 
│   │   └── CloudUserViewModel.swift
│   └── UI/
│       ├── Components/
│       │   ├── UserRowView.swift
│       │   ├── UserRowView.swift
│       │   └── UserRowView.swift
│       └── UserListView.swift
└── UserApp.swift
```

---

#### App Entry Point

<details>

<summary>UserApp.swift</summary>

```
@main
struct UserApp: App {
  var body: some Scene {
    WindowGroup {
      UserListView()
    }
  }
}

#Preview {
  UserListView()
}
```

</details>

---

#### UI Layer

<details>

<summary>Feature/UI/UserListView.swift</summary>

```
import SwiftUI

struct UserListView: View {
  @StateObject private var viewModel = CloudUserViewModel()
  
  var body: some View {
    NavigationView {
      ZStack {
        if viewModel.isLoading {
          ProgressView("Loading users...")
        } else if let error = viewModel.errorMessage {
          VStack(spacing: 16) {
            Image(systemName: "exclamationmark.triangle")
              .font(.system(size: 48))
              .foregroundColor(.orange)
            Text("Error")
              .font(.headline)
            Text(error)
              .font(.subheadline)
              .foregroundColor(.secondary)
              .multilineTextAlignment(.center)
              .padding(.horizontal)
            Button("Retry") {
              Task {
                await viewModel.loadUsers()
              }
            }
            .buttonStyle(.borderedProminent)
          }
        } else {
          List(viewModel.filteredUsers) { user in
            NavigationLink(destination: UserDetailView(user: user)) {
              UserRowView(user: user)
            }
          }
          .searchable(text: $viewModel.searchText, prompt: "Search users")
        }
      }
      .navigationTitle("Users")
      .toolbar {
        ToolbarItem(placement: .navigationBarTrailing) {
          Button {
            Task {
              await viewModel.syncUsers()
            }
          } label: {
            Image(systemName: "arrow.clockwise")
          }
          .disabled(viewModel.isLoading)
        }
      }
    }
    .task {
      await viewModel.loadUsers()
    }
  }
}
```

</details>

<details>

<summary>Feature/UI/Components/UserRowView.swift</summary>

```
import SwiftUI

struct UserRowView: View {
  let user: UserEntity
  
  var body: some View {
    VStack(alignment: .leading, spacing: 8) {
      Text(user.name)
        .font(.headline)
      
      HStack {
        Image(systemName: "person.circle")
          .foregroundColor(.blue)
        Text(user.username)
          .font(.subheadline)
          .foregroundColor(.secondary)
      }
      
      HStack {
        Image(systemName: "building.2")
          .foregroundColor(.purple)
        Text(user.company)
          .font(.subheadline)
          .foregroundColor(.secondary)
      }
    }
    .padding(.vertical, 4)
  }
}
```

</details>

<details>

<summary>Feature/UI/Components/UserDetailView.swift</summary>

```
import SwiftUI

struct UserDetailView: View {
  let user: UserEntity
  
  var body: some View {
    List {
      Section("Personal Information") {
        DetailRow(icon: "person.fill", title: "Name", value: user.name)
        DetailRow(icon: "at", title: "Username", value: user.username)
        DetailRow(icon: "envelope.fill", title: "Email", value: user.email)
        DetailRow(icon: "phone.fill", title: "Phone", value: user.phone)
        DetailRow(icon: "globe", title: "Website", value: user.website)
      }
      
      Section("Location") {
        DetailRow(icon: "location.fill", title: "City", value: user.city)
      }
      
      Section("Company") {
        DetailRow(icon: "building.2.fill", title: "Company", value: user.company)
        VStack(alignment: .leading, spacing: 4) {
          HStack {
            Image(systemName: "quote.opening")
              .foregroundColor(.blue)
            Text("Catch Phrase")
              .font(.subheadline)
              .foregroundColor(.secondary)
          }
          Text(user.catchPhrase)
            .font(.body)
            .italic()
            .padding(.leading, 24)
        }
      }
    }
    .navigationTitle(user.name)
    .navigationBarTitleDisplayMode(.inline)
  }
}
```

</details>

<details>

<summary>Feature/UI/Components/DetailRow.swift</summary>

```
import SwiftUI

struct DetailRow: View {
  let icon: String
  let title: String
  let value: String
  
  var body: some View {
    HStack {
      Image(systemName: icon)
        .foregroundColor(.blue)
        .frame(width: 24)
      VStack(alignment: .leading, spacing: 2) {
        Text(title)
          .font(.caption)
          .foregroundColor(.secondary)
        Text(value)
          .font(.body)
      }
    }
  }
}
```

</details>

---

#### Presentation Layer

<details>

<summary>Feature/Presentation/CloudUserViewModel.swift</summary>

```
import Combine
import Foundation

class CloudUserViewModel: ObservableObject {
  @Published var users: [UserEntity] = []
  @Published var isLoading = false
  @Published var errorMessage: String?
  @Published var searchText = ""
  
  private let getUsersUseCase = CloudUserUseCaseImpl()
  private let syncUseCase = CloudSyncUseCaseImpl()
  
  var filteredUsers: [UserEntity] {
    if searchText.isEmpty {
      return users
    }
    return users.filter { user in
      user.name.localizedCaseInsensitiveContains(searchText) ||
      user.username.localizedCaseInsensitiveContains(searchText) ||
      user.email.localizedCaseInsensitiveContains(searchText) ||
      user.company.localizedCaseInsensitiveContains(searchText)
    }
  }
  
  func loadUsers() async {
    isLoading = true
    errorMessage = nil
    
    do {
      users = try await getUsersUseCase.execute()
    } catch {
      errorMessage = error.localizedDescription
    }
    
    isLoading = false
  }
  
  func syncUsers() async {
    isLoading = true
    errorMessage = nil
    
    do {
      try await syncUseCase.execute()
      users = try await getUsersUseCase.execute()
    } catch {
      errorMessage = error.localizedDescription
    }
    
    isLoading = false
  }
}
```

</details>

---

#### Domain Layer

<details>

<summary>Feature/Domain/Entities/UserEntity.swift</summary>

```
struct UserEntity: Identifiable {
  let id: Int
  let name: String
  let username: String
  let email: String
  let city: String
  let company: String
  let phone: String
  let website: String
  let catchPhrase: String
}
```

</details>

<details>

<summary>Feature/Domain/UseCases/CloudUserUseCase.swift</summary>

```
protocol CloudUserUseCase {
  func execute() async throws -> [UserEntity]
}

class CloudUserUseCaseImpl: CloudUserUseCase {
  private let service: CloudApiService
  
  init(service: CloudApiService = CloudApiServiceImpl()) {
    self.service = service
  }
  
  func execute() async throws -> [UserEntity] {
    let responses = try await service.getUsers()
    return responses.map { response in
      UserEntity(
        id: response.id,
        name: response.name,
        username: response.username,
        email: response.email,
        city: response.address.city,
        company: response.company.name,
        phone: response.phone,
        website: response.website,
        catchPhrase: response.company.catchPhrase
      )
    }
  }
}
```

</details>

<details>

<summary>Feature/Domain/UseCases/CloudSyncUseCase.swift</summary>

```
protocol CloudSyncUseCase {
  func execute() async throws
}

class CloudSyncUseCaseImpl: CloudSyncUseCase {
  private let service: CloudApiService
  
  init(service: CloudApiService = CloudApiServiceImpl()) {
    self.service = service
  }
  
  func execute() async throws {
    try await service.syncUsers()
  }
}
```

</details>

---

#### Data Layer

<details>

<summary>Feature/Data/Network/CloudUserRequest.swift</summary>

```
struct CloudUserRequest: Codable {
  var endpoint: String = "/users"
}
```

</details>

<details>

<summary>Feature/Data/Network/CloudUserResponse.swift</summary>

```
struct CloudUserResponse: Codable {
  let id: Int
  let name: String
  let username: String
  let email: String
  let address: Address
  let phone: String
  let website: String
  let company: Company
  
  struct Address: Codable {
    let street: String
    let suite: String
    let city: String
    let zipcode: String
    let geo: Geo
    
    struct Geo: Codable {
      let lat: String
      let lng: String
    }
  }
  
  struct Company: Codable {
    let name: String
    let catchPhrase: String
    let bs: String
  }
}
```

</details>

<details>

<summary>Feature/Data/Network/CloudUserApi.swift</summary>

```
import Foundation

protocol CloudUserApi {
  func fetchUsers() async throws -> [CloudUserResponse]
}

class CloudUserApiImpl: CloudUserApi {
  private let baseURL = "https://jsonplaceholder.typicode.com"
  
  func fetchUsers() async throws -> [CloudUserResponse] {
    guard let url = URL(string: "\(baseURL)/users") else {
      throw URLError(.badURL)
    }
    
    let (data, _) = try await URLSession.shared.data(from: url)
    let users = try JSONDecoder().decode([CloudUserResponse].self, from: data)
    return users
  }
}
```

</details>


<details>

<summary>Feature/Data/Service/CloudApiService.swift</summary>

```
protocol CloudApiService {
  func getUsers() async throws -> [CloudUserResponse]
  func syncUsers() async throws
}

class CloudApiServiceImpl: CloudApiService {
  private let api: CloudUserApi
  
  init(api: CloudUserApi = CloudUserApiImpl()) {
    self.api = api
  }
  
  func getUsers() async throws -> [CloudUserResponse] {
    return try await api.fetchUsers()
  }
  
  func syncUsers() async throws {
    _ = try await api.fetchUsers()
  }
}

```

</details>




