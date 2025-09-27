
---

# ✅ Checklist rápido – Personagem modular no Blender

1. **Preparar base**

   * `Ctrl + A → Apply All Transforms`
   * Conferir Armature em rest pose
   * Nomes de ossos organizados

2. **Importar peças**

   * `File → Append → Object`
   * `Alt + G`, `Alt + R`, `Alt + S` (reset)
   * `Ctrl + A → Apply Rotation & Scale`

3. **Vincular ao Armature**

   * Cabelo/Acessório → `Ctrl + P → With Automatic Weights`
   * Roupa → `Modifier → Data Transfer (Vertex Groups)` → Apply → Add Armature
   * Armadura rígida/arma → `Ctrl + P → Bone`

4. **Ajustar pesos**

   * `Weight Paint` → Normalize / Clean se necessário

5. **Organizar Outliner**

   ```
   Armature
    ├─ Base_Body
    ├─ SLOT_Hair
    ├─ SLOT_Armor
    └─ SLOT_Weapon
   ```

6. **Animações**

   * Actions → `Push Down` no NLA Editor
   * Renomear strips (Idle, Run, Jump, etc.)

7. **Exportar GLB**

   * `File → Export → glTF 2.0 (.glb)`
   * Format: GLB
   * Include: Selected Objects
   * Geometry: Apply Modifiers, UVs, Normals
   * Animation: NLA Strips, Animations ON

---

