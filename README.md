# SwiftUI-Pager

```swift
import SwiftUI

struct ContentView: View {
    @State private var currentPageIndex = 0
    
    var body: some View {
        VerticalPageViewController(pages(), currentPageIndex: self.$currentPageIndex)
        .frame(height: UIScreen.main.bounds.height)
        .ignoresSafeArea()
    }
    
    func pages() -> [AnyView] {
        [
            ZStack {
                Color.red
                Text("Page 1")
            }.frame(maxWidth: .infinity, maxHeight: .infinity)
                .ignoresSafeArea()
                .eraseToAnyView(),
            ZStack {
                Color.green
                Text("Page 2")
            }.frame(maxWidth: .infinity, maxHeight: .infinity)
                .ignoresSafeArea()
                .eraseToAnyView(),
            ZStack {
                Color.blue
                Text("Page 3")
            }.frame(maxWidth: .infinity, maxHeight: .infinity)
                .ignoresSafeArea()
                .eraseToAnyView()
            
        ]
        
    }

}

extension View {
    
    func eraseToAnyView() -> AnyView {
        AnyView(self)
    }
    
}


struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

struct VerticalPageViewController<Page: View>: UIViewControllerRepresentable {
    var pages: [Page]
    @Binding var currentPageIndex: Int

    init(_ pages: [Page], currentPageIndex: Binding<Int>) {
        self.pages = pages
        self._currentPageIndex = currentPageIndex
    }

    func makeUIViewController(context: Context) -> UIPageViewController {
        let pageViewController = UIPageViewController(
            transitionStyle: .scroll,
            navigationOrientation: .vertical)

        pageViewController.dataSource = context.coordinator
        pageViewController.delegate = context.coordinator

        return pageViewController
    }

    func updateUIViewController(_ pageViewController: UIPageViewController, context: Context) {
        pageViewController.setViewControllers(
            [context.coordinator.controllers[currentPageIndex]],
            direction: .forward,
            animated: true)
    }

    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    class Coordinator: NSObject, UIPageViewControllerDataSource, UIPageViewControllerDelegate {
        var parent: VerticalPageViewController
        var controllers = [UIViewController]()

        init(_ pageViewController: VerticalPageViewController) {
            self.parent = pageViewController
            controllers = parent.pages.map { UIHostingController(rootView: $0) }
        }

        func pageViewController(_ pageViewController: UIPageViewController, viewControllerBefore viewController: UIViewController) -> UIViewController? {
            guard let index = controllers.firstIndex(of: viewController) else {
                return nil
            }
            if index == 0 {
                return controllers.last
            }
            return controllers[index - 1]
        }

        func pageViewController(_ pageViewController: UIPageViewController, viewControllerAfter viewController: UIViewController) -> UIViewController? {
            guard let index = controllers.firstIndex(of: viewController) else {
                return nil
            }
            if index + 1 == controllers.count {
                return controllers.first
            }
            return controllers[index + 1]
        }

        func pageViewController(_ pageViewController: UIPageViewController, didFinishAnimating finished: Bool, previousViewControllers: [UIViewController], transitionCompleted completed: Bool) {
            if completed,
               let visibleViewController = pageViewController.viewControllers?.first,
               let index = controllers.firstIndex(of: visibleViewController) {
                parent.currentPageIndex = index
            }
        }
    }
}

```
