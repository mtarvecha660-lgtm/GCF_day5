# GCF_day5
import SwiftUI
import SpriteKit

// MARK: - 1. The Game Scene (Physics & Logic)
class CircuitScene: SKScene {
    
    var electron: SKShapeNode?
    var inputSlot: SKShapeNode? // The "Magnetic" target
    var isDragging = false
    
    override func didMove(to view: SKView) {
        // VISUALS: Dark PCB Green
        self.backgroundColor = SKColor(red: 0.05, green: 0.25, blue: 0.1, alpha: 1.0)
        
        // BORDERS: So the electron stays on screen
        let border = SKPhysicsBody(edgeLoopFrom: self.frame)
        border.friction = 0.0
        border.restitution = 0.5
        self.physicsBody = border
        self.physicsWorld.gravity = CGVector(dx: 0, dy: 0) // Zero gravity
        
        // Create the elements
        createInputSlot()
        createElectron()
    }
    
    // Create the "Hole" where the electron should go (The Logic Gate Input)
    func createInputSlot() {
        // A simple hollow circle representing a pin on a chip
        let slot = SKShapeNode(circleOfRadius: 30)
        slot.position = CGPoint(x: size.width / 2, y: size.height * 0.7) // Top center
        slot.strokeColor = .gold // Custom color helper defined below
        slot.lineWidth = 3
        slot.fillColor = SKColor.black.withAlphaComponent(0.5)
        slot.name = "inputSlot"
        
        // Label it
        let label = SKLabelNode(text: "INPUT A")
        label.fontSize = 12
        label.fontColor = .gold
        label.position = CGPoint(x: 0, y: 40)
        slot.addChild(label)
        
        addChild(slot)
        self.inputSlot = slot
    }
    
    func createElectron() {
        let node = SKShapeNode(circleOfRadius: 20)
        
        // VISUALS: Glowing Electric Blue
        node.fillColor = .cyan
        node.strokeColor = .white
        node.glowWidth = 5.0
        node.position = CGPoint(x: size.width / 2, y: size.height * 0.2) // Start at bottom
        node.name = "electron"
        
        // PHYSICS
        node.physicsBody = SKPhysicsBody(circleOfRadius: 20)
        node.physicsBody?.allowsRotation = true
        node.physicsBody?.mass = 0.5
        node.physicsBody?.linearDamping = 2.0 // Air resistance (stops it from floating forever)
        
        addChild(node)
        self.electron = node
    }
    
    // MARK: - Interaction Handling
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let touch = touches.first else { return }
        let location = touch.location(in: self)
        
        if let node = atPoint(location) as? SKShapeNode, node.name == "electron" {
            isDragging = true
            node.physicsBody?.velocity = .zero // Stop moving when grabbed
            node.run(SKAction.scale(to: 1.2, duration: 0.1)) // Pop effect
        }
    }
    
    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard isDragging, let touch = touches.first, let node = electron else { return }
        let location = touch.location(in: self)
        
        // Move directly to finger
        node.position = location
    }
    
    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard isDragging, let node = electron else { return }
        isDragging = false
        node.run(SKAction.scale(to: 1.0, duration: 0.1))
        
        // CHECK MAGNETISM
        checkForMagneticSnap()
    }
    
    // MARK: - The "Snap" Logic
    func checkForMagneticSnap() {
        guard let electron = electron, let slot = inputSlot else { return }
        
        // Calculate distance between Electron and Slot
        let distance = hypot(electron.position.x - slot.position.x, electron.position.y - slot.position.y)
        
        // If within 50 pixels... SNAP!
        if distance < 50 {
            let snapAction = SKAction.move(to: slot.position, duration: 0.1)
            let soundAction = SKAction.playSoundFileNamed("click", waitForCompletion: false) // Needs a file, or it just skips
            
            // Visual feedback: Turn the slot green to show it's "Active"
            let activateSlot = SKAction.run {
                slot.strokeColor = .green
                slot.fillColor = SKColor.green.withAlphaComponent(0.2)
            }
            
            // Run the sequence
            electron.run(SKAction.group([snapAction]))
            slot.run(activateSlot)
            
            // Lock the electron so it doesn't drift away
            electron.physicsBody?.isDynamic = false
        }
    }
}

// Helper for colors
extension UIColor {
    static let gold = UIColor(red: 1.0, green: 0.84, blue: 0.0, alpha: 1.0)
}

// MARK: - 2. The SwiftUI View
struct ContentView: View {
    
    // Create the scene
    var scene: SKScene {
        let scene = CircuitScene()
        scene.size = CGSize(width: 400, height: 800)
        scene.scaleMode = .aspectFill
        return scene
    }

    var body: some View {
        ZStack {
            // Background color for the edges of the iPhone
            Color(red: 0.05, green: 0.25, blue: 0.1)
                .ignoresSafeArea()
            
            // The SpriteKit View
            SpriteView(scene: scene)
                .frame(maxWidth: .infinity, maxHeight: .infinity)
                .ignoresSafeArea()
        }
    }
}

#Preview {
    ContentView()
}
