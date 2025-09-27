Show 👍 vou te montar uma **lista de passos bem detalhada, mas simplificada** de como preparar no **Blender** seus personagens modulares para exportar em **glTF/GLB** e usar no **Three.js**.
Incluí também **opções extras** que você pode escolher dependendo do tipo de objeto (cabelo, roupa, acessório, corpo etc).

---

# 📋 Passos no Blender (para personagens modulares)

### 🔹 1. Preparar o modelo base

* Abra seu arquivo com **Armature** + **mesh base**.
* Certifique-se de que:

  * Escala = 1, Rotação = 0 (use `Ctrl + A → Apply All Transforms`).
  * O Armature está em **rest pose**.
  * Os nomes dos ossos estão organizados (sem duplicados estranhos).

---

### 🔹 2. Importar peças extras

* Vá em `File → Append` → abra o arquivo da peça (cabelo, roupa, acessório).
* Entre em `Object` e selecione apenas o objeto da peça (não traga outro armature).
* Posicione no lugar correto (`G` mover, `R` rotacionar, `S` escalar).

👉 **Extra:** se a peça vier fora de escala/rotação, selecione ela e faça:

* `Ctrl + A → Apply Rotation & Scale`
* `Alt + G` (reset position), `Alt + R` (reset rotation), `Alt + S` (reset scale)

---

### 🔹 3. Vincular ao esqueleto

Agora você precisa que a peça siga o mesmo Armature do corpo.
Duas opções:

**Opção A (rápido, bom para cabelo e acessórios):**

1. Selecione a peça, depois `Shift` + selecione o Armature.
2. `Ctrl + P → With Automatic Weights`.
3. Teste em `Pose Mode`.

**Opção B (mais preciso, bom para roupas):**

1. Selecione a peça.
2. Adicione o modificador **Data Transfer**.
3. Em *Source*, escolha a mesh base.
4. Ative *Vertex Groups* → *Nearest Face Interpolated*.
5. Clique **Apply**.
6. Adicione um modificador **Armature** apontando para o Armature.
7. Teste em `Pose Mode`.

---

### 🔹 4. Ajustar pesos

* Entre em **Weight Paint** se algo deformar errado.
* Use `Normalize All` para equilibrar pesos.
* `Clean` remove pesos inúteis.

---

### 🔹 5. Organizar por slots

No **Outliner**, organize assim:

```
Armature
 ├─ Base_Body
 ├─ SLOT_Hair
 │    ├─ Hair_01
 │    └─ Hair_02
 ├─ SLOT_Armor
 │    ├─ Armor_01
 │    └─ Armor_02
```

👉 Isso facilita ativar/desativar partes no Three.js (via `object.visible = true/false`).

---

### 🔹 6. Animações

* Coloque cada ação (Idle, Run, Jump) no **NLA Editor** (`Push Down Action`).
* Renomeie cada strip para nomes fáceis (ex: "Idle", "Run").

---

### 🔹 7. Exportar para GLTF/GLB

* `File → Export → glTF 2.0 (.glb)`
* **Formato**: GLB (binário, mais prático).
* **Include**: `Selected Objects` (selecione Armature + meshes).
* **Geometry**: Apply Modifiers, Normals ON, UVs ON.
* **Animation**: marque Animations, NLA Strips → ON.
* Exporte.

---

# ⚡ Extras por tipo de objeto

* **Cabelo/Acessórios:**

  * Normalmente só precisam do bone da cabeça.
  * Parent + Automatic Weights já basta.

* **Roupas:**

  * Melhor usar **Data Transfer** de pesos do corpo para a roupa.
  * Cheque clipping (pode precisar ajustar mesh manualmente).

* **Armadura rígida (sem deformação):**

  * Pode só parentear ao osso (sem pesos).
  * `Ctrl + P → Bone`.

* **Armas na mão:**

  * Parent direto ao bone da mão.

---

# 🔹 Resumindo

1. Apply Transforms em tudo.
2. Importar peças → alinhar → aplicar rotação/escala.
3. Vincular ao Armature (Automatic Weights ou Transfer).
4. Ajustar pesos se precisar.
5. Organizar Outliner em "slots".
6. Colocar ações no NLA Editor.
7. Exportar para `.glb` com animações.

---

Quer que eu faça também uma **versão curta estilo checklist rápido** (sem explicações, só ações na ordem certa) pra você seguir como receita?
