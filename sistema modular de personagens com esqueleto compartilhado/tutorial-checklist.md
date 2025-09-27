
* **Parte 1 → Blender (preparar base e partes separadas)**
* **Parte 2 → Three.js (carregar e reusar o esqueleto do base)**

---

# ✅ Checklist Blender (modelo base + partes separadas)

### 🔹 Base (modelo-base.glb)

1. Deixe só o **Armature** + **mesh base** (corpo, cabeça, braços, pernas).
2. `Ctrl + A → Apply All Transforms` em tudo.
3. No **NLA Editor**, jogue as animações (Idle, Run, Jump...).
4. `File → Export → glTF 2.0 (.glb)` → nomeie `modelo-base.glb`.

---

### 🔹 Partes separadas (cabelo, roupa, acessório)

1. Abra o arquivo com a peça.
2. **Importe o Armature do base** (Append) → use o mesmo Armature do modelo-base.

   * ❌ Não crie armature novo.
3. Ajuste a peça (posição, escala).

   * `Alt + G/R/S` para reset
   * `Ctrl + A → Apply Rotation & Scale`
4. Vincule a peça ao **mesmo Armature**:

   * Cabelo/acessórios → `Ctrl + P → With Automatic Weights`.
   * Roupa → `Modifier → Data Transfer (Vertex Groups)` → Apply → Add Armature.
   * Armadura rígida/arma → `Ctrl + P → Bone`.
5. Delete o corpo, mantenha só a peça + Armature.
6. Exporte como `.glb` → ex: `hair_01.glb`, `armor_01.glb`.

👉 Importante: os ossos do Armature precisam ter **nomes idênticos** ao modelo-base.

---

# ✅ Checklist Three.js (usar modelo-base + partes extras)

1. **Carregar o modelo-base**

```js
const loader = new GLTFLoader();
loader.load('modelo-base.glb', (gltf) => {
  const model = gltf.scene;
  scene.add(model);

  const baseSkinnedMesh = model.getObjectByProperty('type', 'SkinnedMesh');
  const skeleton = baseSkinnedMesh.skeleton;

  // guardar skeleton para reusar
});
```

2. **Carregar uma parte extra**

```js
loader.load('hair_01.glb', (gltf) => {
  const hair = gltf.scene.getObjectByProperty('type', 'SkinnedMesh');

  // reusar o skeleton do modelo-base
  hair.skeleton = skeleton;
  hair.bind(skeleton, hair.bindMatrix);

  // adicionar ao personagem
  model.add(hair);
});
```

3. **Trocar peça (ex.: cabelo)**

```js
// remove o cabelo atual
model.remove(currentHair);

// carrega outro .glb e adiciona
loader.load('hair_02.glb', (gltf) => {
  const hair = gltf.scene.getObjectByProperty('type', 'SkinnedMesh');
  hair.skeleton = skeleton;
  hair.bind(skeleton, hair.bindMatrix);
  model.add(hair);
  currentHair = hair;
});
```

4. **Animações (no modelo-base)**

```js
const mixer = new THREE.AnimationMixer(model);
const idle = gltf.animations.find(a => a.name === 'Idle');
mixer.clipAction(idle).play();
```

👉 As partes extras vão seguir o esqueleto automaticamente.

---

⚡ Em resumo:

* No Blender → **sempre use o mesmo Armature** em todas as partes antes de exportar.
* No Three.js → **carregue o skeleton do modelo-base** e reatribua para cada parte carregada separadamente.

---


