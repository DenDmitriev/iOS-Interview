**Frame** — задается относительно собственного superview, 
**Bounds** — относительно собственной координатной системы.

```swift
let view = UIView()
view.frame = CGRect(x: 100, y: 0, width: 50, height: 50)

print("frame", view.frame) // frame (100.0, 0.0, 50.0, 50.0)
print("bounds", view.bounds) // bounds (0.0, 0.0, 50.0, 50.0)
```
