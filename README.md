# Login Challenge

## Yuta Koshizawa **@koher**

---

## 目標

- コードの見通しを良くする
    - View からロジックを分離
    - 状態遷移をわかりやすく
    - 依存関係をわかりやすく
- コードの品質を担保する
    - 単体テストの導入

---

## Login ではなく Home について説明します🙏

---

![inline](img/home-view.png)

---

## View からロジックを分離 (Before)

```swift
struct HomeView: View {
    @State private var user: User?
    ...
    var body: some View {
        ...
        Text(user?.name ?? "User Name")
        ...
    }
    ...
}
```

View の中に状態が記述されている。

---

## View からロジックを分離

---

## View からロジックを分離 (Before)

```swift
do {
    // API を叩いて User を取得。
    let user = try await UserService.currentUser()

    // 取得した情報を View に反映。
    self.user = user
} catch ... {
    ...
}
```

画面表示時に View から API を叩いて `user` を更新している。

---

## View からロジックを分離 (Before)

```swift
struct HomeView: View {
    @State private var user: User?
    ...
    var body: some View {
        ...
        Text(user?.name ?? "User Name")
        ...
    }
    ...
}
```

---

## View からロジックを分離 (After)

```swift
struct HomeView: View {
    @StateObject private var state: HomeViewState
    ...
    var body: some View {
        ...
        Text(state.user?.name ?? "User Name")
        ...
    }
    ...
}
```

ViewModel 的なクラスに状態を保持させる。

---

## View からロジックを分離 (After)

```swift
@MainActor
public final class HomeViewState: ObservableObject {
    @Published public var user: User?
    ...
}
```

ViewModel 的なクラスに状態を保持させる。

---

## View からロジックを分離 (After)

```swift
@MainActor
public final class HomeViewState: ObservableObject {
    @Published public private(set) var user: User?
    ...
}
```

`user` は内部からしか更新しないので `private(set)` に。

---

## View からロジックを分離 (After)

```swift
public func loadUser() async {
    ...
    do {
        // API を叩いて User を取得。
        let user = try await UserService.currentUser()

        // 取得した情報を View に反映。
        self.user = user
    } catch ... { ... }
}
```

`user` をロードするロジックも `state` 側に記述する。

---

## View からロジックを分離 (After)

```swift
@MainActor
public final class HomeViewState: ObservableObject {
    @Published public private(set) var user: User?
    ...
}
```

`user` は `@Published` なので、 `user` が更新されると `objectWillChange` が発火して View に反映される。

---

## View からロジックを分離 (After)

```swift
@MainActor
public final class HomeViewState: ObservableObject {
    ...
    @Published public var presentsNetworkErrorAlert
        : Bool　= false
    ...
}
```

`loadUser` のエラーハンドリングでネットワークエラーのアラートを表示する。

---

## View からロジックを分離 (After)

```swift
public func loadUser() async {
    ...
    do {
        ...
    } catch let error as NetworkError {
        ...
        presentsNetworkErrorAlert = true
    } ...
}
```

`loadUser` のエラーハンドリングでネットワークエラーのアラートを表示する。

---

## 状態遷移をわかりやすく

---

