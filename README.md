import SwiftUI
import SpriteKit
import SceneKit

// MARK: - MAIN CONTENT VIEW (State Manager)
struct ContentView: View {
    @State private var currentLevel = 1
    @State private var levelComplete = false
    @State private var showVictory = false
    
    var body: some View {
        ZStack {
            // Background
            Color.black.ignoresSafeArea()
            
            // LEVEL SWITCHER
            if currentLevel == 1 {
                // LEVEL 1: THE TRANSISTOR
                Level1View(onComplete: {
                    withAnimation { levelComplete = true }
                })
                .transition(.opacity)
                
            } else if currentLevel == 2 {
                // LEVEL 2: THE CPU
                Level2View(onComplete: {
                    withAnimation { levelComplete = true }
                })
                .transition(.opacity)
                
            } else if currentLevel == 3 {
                // LEVEL 3: THE GPU
                GPUView(onComplete: {
                    withAnimation { showVictory = true }
                })
                .transition(.opacity)
            }
            
            // "NEXT LEVEL" BUTTON
            if levelComplete {
                VStack {
                    Spacer()
                    Button(action: {
                        withAnimation {
                            currentLevel += 1
                            levelComplete = false
                        }
                    }) {
                        HStack {
                            Image(systemName: "arrow.right.circle.fill")
                            Text(buttonTitle)
                                .bold()
                        }
                        .padding()
                        .frame(width: 280)
                        .background(currentLevel == 1 ? Color.green : Color.purple)
                        .foregroundColor(.white)
                        .cornerRadius(20)
                        .shadow(radius: 10)
                    }
                    .padding(.bottom, 50)
                    .transition(.move(edge: .bottom))
                }
            }
            
            // VICTORY SCREEN
            if showVictory {
                ZStack {
                    Color.black.opacity(0.9).ignoresSafeArea()
                    VStack(spacing: 20) {
                        Text("SYSTEM BOOTED")
                            .font(.largeTitle)
                            .bold()
                            .foregroundColor(.green)
                        Text("You just simulated the journey from a single bit of electricity to a 3D simulation.")
                            .multilineTextAlignment(.center)
                            .foregroundColor(.gray)
                            .padding()
                        Button("Reboot") {
                            currentLevel = 1
                            showVictory = false
                        }
                        .padding()
                        .background(Color.white)
                        .cornerRadius(10)
                    }
                }
                .transition(.opacity)
            }
        }
    }
    
    var buttonTitle: String {
        switch currentLevel {
        case 1: return "ZOOM OUT TO CPU"
        case 2: return "LAUNCH GPU RENDER"
        default: return "NEXT"
        }
    }
}

// MARK: - LEVEL 1: CIRCUIT SCENE (Physics)
struct Level1View: View {
    var onComplete: () -> Void
    
    var scene: SKScene {
        let scene = CircuitScene()
        scene.size = CGSize(width: 400, height: 850)
        scene.scaleMode = .aspectFill
        scene.onGateActivated = onComplete
        return scene
    }
    
    var body: some View {
        SpriteView(scene: scene)
            .ignoresSafeArea()
            .overlay(
                VStack {
                    Text("LEVEL 1: THE LOGIC")
                        .font(.headline)
                        .foregroundColor(.green)
                        .padding(.top, 60)
                    Text("Drag electrons to inputs to fire the gate.")
                        .font(.caption)
                        .foregroundColor(.gray)
                    Spacer()
                }
            )
    }
}

class CircuitScene: SKScene {
    var onGateActivated: (() -> Void)?
    var inputSlots: [SKShapeNode] = []
    var outputSlot: SKShapeNode?
    var slotA_Filled = false
    var slotB_Filled = false
    var isDragging = false
    var draggedNode: SKShapeNode?
    
    override func didMove(to view: SKView) {
        backgroundColor = SKColor(red: 0.05, green: 0.25, blue: 0.1, alpha: 1.0)
        physicsWorld.gravity = .zero
        
        // Setup Visuals
        createChipBody()
        createInputSlot(name: "SlotA", position: CGPoint(x: size.width * 0.3, y: size.height * 0.6))
        createInputSlot(name: "SlotB", position: CGPoint(x: size.width * 0.7, y: size.height * 0.6))
        createOutputSlot(position: CGPoint(x: size.width * 0.5, y: size.height * 0.85))
        
        // Spawn Electrons
        spawnElectron(at: CGPoint(x: size.width * 0.3, y: size.height * 0.2))
        spawnElectron(at: CGPoint(x: size.width * 0.7, y: size.height * 0.2))
    }
    
    func createChipBody() {
        let chip = SKShapeNode(rectOf: CGSize(width: 220, height: 220), cornerRadius: 10)
        chip.position = CGPoint(x: size.width/2, y: size.height * 0.72)
        chip.fillColor = SKColor.black.withAlphaComponent(0.6)
        chip.strokeColor = .gray
        chip.zPosition = -1
        addChild(chip)
        
        let label = SKLabelNode(text: "AND GATE")
        label.fontSize = 20; label.fontName = "Courier-Bold"; label.fontColor = .white
        chip.addChild(label)
    }
    
    func createInputSlot(name: String, position: CGPoint) {
        let slot = SKShapeNode(circleOfRadius: 25)
        slot.position = position
        slot.strokeColor = .yellow
        slot.lineWidth = 3
        slot.name = name
        
        let label = SKLabelNode(text: name == "SlotA" ? "A" : "B")
        label.fontSize = 14; label.fontColor = .yellow; label.position = CGPoint(x: 0, y: -45)
        slot.addChild(label)
        addChild(slot)
        inputSlots.append(slot)
    }
    
    func createOutputSlot(position: CGPoint) {
        let slot = SKShapeNode(circleOfRadius: 25)
        slot.position = position
        slot.strokeColor = .gray
        slot.lineWidth = 3
        slot.name = "Output"
        
        let label = SKLabelNode(text: "OUT")
        label.fontSize = 14; label.fontColor = .white; label.position = CGPoint(x: 0, y: 45)
        slot.addChild(label)
        outputSlot = slot
    }
    
    func spawnElectron(at position: CGPoint) {
        let node = SKShapeNode(circleOfRadius: 20)
        node.fillColor = .cyan; node.strokeColor = .white; node.glowWidth = 5.0
        node.position = position; node.name = "electron"
        node.physicsBody = SKPhysicsBody(circleOfRadius: 20)
        node.physicsBody?.allowsRotation = true
        node.physicsBody?.mass = 0.5; node.physicsBody?.linearDamping = 1.0
        addChild(node)
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let touch = touches.first else { return }
        let location = touch.location(in: self)
        let nodes = nodes(at: location)
        for node in nodes {
            if node.name == "electron", node.physicsBody?.isDynamic == true {
                draggedNode = node as? SKShapeNode
                isDragging = true
                draggedNode?.run(SKAction.scale(to: 1.2, duration: 0.1))
                draggedNode?.physicsBody?.velocity = .zero
            }
        }
    }
    
    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard isDragging, let touch = touches.first, let node = draggedNode else { return }
        node.position = touch.location(in: self)
    }
    
    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard isDragging, let node = draggedNode else { return }
        isDragging = false
        node.run(SKAction.scale(to: 1.0, duration: 0.1))
        checkMagneticSnap(for: node)
        draggedNode = nil
    }
    
    func checkMagneticSnap(for electron: SKShapeNode) {
        for slot in inputSlots {
            if hypot(electron.position.x - slot.position.x, electron.position.y - slot.position.y) < 50 {
                electron.run(SKAction.move(to: slot.position, duration: 0.1))
                electron.physicsBody?.isDynamic = false
                slot.strokeColor = .green; slot.fillColor = SKColor.green.withAlphaComponent(0.3)
                if slot.name == "SlotA" { slotA_Filled = true }
                if slot.name == "SlotB" { slotB_Filled = true }
                if slotA_Filled && slotB_Filled { fireOutput() }
                break
            }
        }
    }
    
    func fireOutput() {
        guard let output = outputSlot else { return }
        let activate = SKAction.run {
            output.strokeColor = .cyan; output.fillColor = SKColor.cyan.withAlphaComponent(0.5)
        }
        let spawnResult = SKAction.run {
            let res = SKShapeNode(circleOfRadius: 20)
            res.fillColor = .white; res.glowWidth = 10.0; res.position = output.position
            self.addChild(res)
            res.run(SKAction.sequence([SKAction.moveBy(x: 0, y: 300, duration: 0.8), SKAction.fadeOut(withDuration: 0.5), SKAction.removeFromParent()]))
        }
        let notify = SKAction.run { self.onGateActivated?() }
        output.run(SKAction.sequence([SKAction.wait(forDuration: 0.2), activate, spawnResult, SKAction.wait(forDuration: 1.0), notify]))
    }
}

// MARK: - LEVEL 2: CPU SCENE (Instructions)
struct Level2View: View {
    var onComplete: () -> Void
    
    var scene: SKScene {
        let scene = CPUScene()
        scene.size = CGSize(width: 400, height: 850)
        scene.scaleMode = .aspectFill
        scene.onTaskCompleted = onComplete
        return scene
    }
    
    var body: some View {
        SpriteView(scene: scene)
            .ignoresSafeArea()
            .overlay(
                VStack {
                    Text("LEVEL 2: THE CPU")
                        .font(.headline)
                        .foregroundColor(.blue)
                        .padding(.top, 60)
                    Text("Drag the instruction to the ALU to process data.")
                        .font(.caption)
                        .foregroundColor(.gray)
                    Spacer()
                }
            )
    }
}

class CPUScene: SKScene {
    var onTaskCompleted: (() -> Void)?
    var registerAX: SKShapeNode?
    var registerBX: SKShapeNode?
    var aluZone: SKShapeNode?
    var labelAX: SKLabelNode?
    var isDragging = false
    var draggedNode: SKShapeNode?
    
    override func didMove(to view: SKView) {
        backgroundColor = SKColor(red: 0.1, green: 0.1, blue: 0.15, alpha: 1.0)
        createRegister(name: "AX", value: "1", xPos: size.width * 0.3)
        createRegister(name: "BX", value: "1", xPos: size.width * 0.7)
        createALU()
        createInstructionBlock()
    }
    
    func createRegister(name: String, value: String, xPos: CGFloat) {
        let box = SKShapeNode(rectOf: CGSize(width: 80, height: 60), cornerRadius: 5)
        box.position = CGPoint(x: xPos, y: size.height * 0.75)
        box.strokeColor = .green; box.fillColor = .black
        
        let valLabel = SKLabelNode(text: value)
        valLabel.fontSize = 24; valLabel.fontName = "Courier-Bold"; valLabel.fontColor = .white
        valLabel.position = CGPoint(x: 0, y: -10)
        box.addChild(valLabel)
        
        let title = SKLabelNode(text: name)
        title.fontSize = 14; title.fontColor = .green; title.position = CGPoint(x: 0, y: 40)
        box.addChild(title)
        
        addChild(box)
        if name == "AX" { registerAX = box; labelAX = valLabel }
        if name == "BX" { registerBX = box }
    }
    
    func createALU() {
        let zone = SKShapeNode(circleOfRadius: 60)
        zone.position = CGPoint(x: size.width/2, y: size.height/2)
        zone.strokeColor = .gray
        zone.lineDashPattern = [10, 5]
        zone.name = "ALU"
        let label = SKLabelNode(text: "ALU")
        label.fontSize = 12; label.fontColor = .gray
        zone.addChild(label)
        addChild(zone)
        aluZone = zone
    }
    
    func createInstructionBlock() {
        let block = SKShapeNode(rectOf: CGSize(width: 160, height: 50), cornerRadius: 8)
        block.position = CGPoint(x: size.width/2, y: size.height * 0.2)
        block.fillColor = .systemBlue; block.strokeColor = .white
        block.name = "Instruction"
        
        let label = SKLabelNode(text: "ADD AX, BX")
        label.fontName = "Courier-Bold"; label.fontSize = 18; label.fontColor = .white
        label.verticalAlignmentMode = .center
        block.addChild(label)
        addChild(block)
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard let touch = touches.first else { return }
        let location = touch.location(in: self)
        if let node = atPoint(location) as? SKShapeNode, node.name == "Instruction" {
            isDragging = true; draggedNode = node
            node.run(SKAction.scale(to: 1.1, duration: 0.1))
        }
    }
    
    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard isDragging, let node = draggedNode, let touch = touches.first else { return }
        node.position = touch.location(in: self)
    }
    
    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        guard isDragging, let node = draggedNode else { return }
        isDragging = false
        node.run(SKAction.scale(to: 1.0, duration: 0.1))
        
        if let alu = aluZone, hypot(node.position.x - alu.position.x, node.position.y - alu.position.y) < 60 {
            performAddition(instruction: node)
        } else {
            node.run(SKAction.move(to: CGPoint(x: size.width/2, y: size.height * 0.2), duration: 0.2))
        }
        draggedNode = nil
    }
    
    func performAddition(instruction: SKShapeNode) {
        guard let alu = aluZone else { return }
        instruction.run(SKAction.move(to: alu.position, duration: 0.1))
        instruction.isUserInteractionEnabled = false
        
        // Animations
        let flash = SKAction.sequence([
            SKAction.wait(forDuration: 0.2),
            SKAction.run { alu.fillColor = .white },
            SKAction.wait(forDuration: 0.1),
            SKAction.run { alu.fillColor = .clear }
        ])
        
        let updateResult = SKAction.run {
            self.labelAX?.text = "2"
            self.labelAX?.fontColor = .cyan
            instruction.removeFromParent()
            self.onTaskCompleted?()
        }
        
        run(SKAction.sequence([flash, SKAction.wait(forDuration: 0.5), updateResult]))
    }
}

// MARK: - LEVEL 3: GPU SCENE (3D Rendering)
struct GPUView: View {
    var onComplete: () -> Void
    @State private var renderProgress: Float = 0.0
    @State private var scene: SCNScene? = nil
    @State private var mainNode: SCNNode? = nil
    
    var body: some View {
        ZStack {
            SceneView(scene: scene, pointOfView: nil, options: [.allowsCameraControl, .autoenablesDefaultLighting])
                .ignoresSafeArea()
                .onAppear(perform: setupScene)
                .onChange(of: renderProgress) { newValue in
                    updateMaterial(value: newValue)
                    if newValue >= 0.95 { onComplete() }
                }
            
            VStack {
                HStack {
                    VStack(alignment: .leading) {
                        Text("GPU MEMORY")
                            .font(.caption).foregroundColor(.gray)
                        Text(renderProgress < 0.5 ? "WIREFRAME" : "ULTRA HD")
                            .font(.title3).bold().foregroundColor(renderProgress < 0.5 ? .green : .purple)
                    }
                    Spacer()
                }
                .padding(.top, 60).padding(.horizontal)
                
                Spacer()
                
                VStack(alignment: .leading) {
                    Text("RENDERING PIPELINE").font(.caption).bold().foregroundColor(.white)
                    HStack {
                        Text("POLY").font(.caption).foregroundColor(.gray)
                        Slider(value: $renderProgress, in: 0...1).accentColor(.purple)
                        Text("HD").font(.caption).foregroundColor(.gray)
                    }
                }
                .padding().background(Color.black.opacity(0.8)).cornerRadius(15).padding(.horizontal).padding(.bottom, 50)
            }
        }
    }
    
    func setupScene() {
        let newScene = SCNScene()
        // NOTE: Replace SCNTorusKnot with SCNScene(named: "car.usdz") for real car
        let geometry = SCNTorusKnot(ringRadius: 10, pipeRadius: 3)
        let node = SCNNode(geometry: geometry)
        
        // Add rotation
        node.runAction(SCNAction.repeatForever(SCNAction.rotateBy(x: 0, y: 1, z: 0.5, duration: 10)))
        newScene.rootNode.addChildNode(node)
        
        self.mainNode = node
        self.scene = newScene
        updateMaterial(value: 0)
    }
    
    func updateMaterial(value: Float) {
        guard let material = mainNode?.geometry?.firstMaterial else { return }
        SCNTransaction.begin()
        SCNTransaction.animationDuration = 0.5
        
        if value < 0.3 {
            material.fillMode = .lines
            material.diffuse.contents = UIColor.green
            material.emission.contents = UIColor.black
            material.metalness.contents = 0.0
        } else if value < 0.7 {
            material.fillMode = .fill
            material.diffuse.contents = UIColor.gray
            material.emission.contents = UIColor.black
            material.metalness.contents = 0.5
        } else {
            material.fillMode = .fill
            material.diffuse.contents = UIColor.systemPurple
            material.emission.contents = UIColor.purple.withAlphaComponent(0.5)
            material.metalness.contents = 1.0
            material.roughness.contents = 0.0
        }
        SCNTransaction.commit()
    }
}
