# iOS 16.2

```swift
struct TimerAttributes: ActivityAttributes {
    public typealias TimerStatus = ContentState
    
    public struct ContentState: Codable, Hashable {
        var endTime: Date
    }
    
    var timerName: String
}

```

```swift
import ActivityKit
import SwiftUI

struct ContentView: View {
    
    @State private var activity: Activity<TimerAttributes>? = nil
    
    var body: some View {
        VStack {
            
            Button("Start Activity") {
                self.startActivity()
            }
            
            Button("Stop Activity") {
                self.stopActivity()
            }
            
        }
        .buttonStyle(.borderedProminent)
        .controlSize(.large)
    }
    
    func startActivity() {
        let attributes = TimerAttributes(timerName: "Dummy Timer")
        let state = TimerAttributes.TimerStatus(endTime: Date().addingTimeInterval(60 * 5))
        
        self.activity = try? Activity<TimerAttributes>.request(attributes: attributes, contentState: state, pushType: .none)
    }
    
    func stopActivity() {
        let state = TimerAttributes.TimerStatus(endTime: Date().addingTimeInterval(60 * 10))
        
        Task {
            await self.activity?.end(using: state, dismissalPolicy: .immediate)
        }
    }
    
    func updateActivity() {
        let state = TimerAttributes.TimerStatus(endTime: Date().addingTimeInterval(60 * 10))
        
        Task {
            await self.activity?.update(using: state)
        }
    }
    
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

```


```swift
import ActivityKit
import WidgetKit
import SwiftUI

struct TimerActivityView: View {
    
    let context: ActivityViewContext<TimerAttributes>
    
    var body: some View {
        VStack {
            Text(self.context.attributes.timerName)
                .font(.headline)
            Text(self.context.state.endTime, style: .timer)
        } //: VStack
        .padding(.horizontal)
    }
    
}

struct TutorialWidget: Widget {
    let kind: String = "TutorialWidget"

    var body: some WidgetConfiguration {
   
        ActivityConfiguration(for: TimerAttributes.self,
                              content: { context in TimerActivityView(context: context) },
                              dynamicIsland: { context in
            
            DynamicIsland {
                DynamicIslandExpandedRegion(.center) {
                    TimerActivityView(context: context)
                }
            } compactLeading: {
                Image(systemName: "circle")
                    .foregroundColor(.green)
            } compactTrailing: {
                Text(context.attributes.timerName)
            } minimal: {
                Image(systemName: "circle")
                    .foregroundColor(.green)
            }
                          
        })
        
    }
}


```
