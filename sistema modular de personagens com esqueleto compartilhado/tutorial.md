Perfeito — vou te mostrar **detalhadamente** o fluxo completo no **Blender** (preparar, riggar/transf. pesos, organizar e exportar) e depois o **exemplo em Three.js** (como carregar e trocar partes no runtime). Vou dar a melhor opção (mais robusta) e uma alternativa avançada para quem quer carregar partes separadas em arquivos diferentes.

# 1) Recomendação principal (mais estável)

**Monte tudo no Blender com um *único armature* e exporte um único `.glb`** contendo:

* o *Armature* + mesh base (com animações)
* todas as peças modulares como *SkinnedMesh* (cada peça é um objeto separado, com seus vertex groups)
  Vantagem: evita retrabalho no runtime, é o método mais confiável para garantir que as animações sigam todas as peças sem remapeamentos complexos.

---

# 2) Passo-a-passo no Blender (detalhado)

### 0) Backup

Faça sempre um backup antes de começar.

### 1 — Preparar a cena base

1. Abra o arquivo com o **modelo base** (mesh dividido em cabeça, tronco, braços, pernas) e o **Armature** já funcionando com as animações.
2. Confirme que o armature está em **rest pose** e que os nomes dos bones estão limpos e únicos (sem espaços estranhos).

### 2 — Importar / Append das partes externas

Se suas partes (cabelo, roupa, acessório) estão em arquivos separados:

* `File → Append` → abra o `.blend` da peça → entre em `Object` e selecione o objeto da mesh (não traga outro armature).
* Posicione a peça no lugar correto (se precisar alinhar, use `G`/`R`/`S`).

### 3 — Fazer a peça usar o mesmo armature (skin)

Existem duas formas principais:

#### Método A — Parent + Automatic Weights (rápido, bom para cabelo simples)

1. Selecione a **peça** (mesh), `Shift` + selecione o **Armature**.
2. `Ctrl + P` → escolha **With Automatic Weights**.
3. Teste em `Pose Mode` (selecione os ossos e mova-os) para ver se a peça segue. Depois refine no Weight Paint.

#### Método B — Transferir pesos do mesh base (mais preciso para roupas que seguem geometria do corpo)

1. Selecione a **peça** (target). Na aba `Modifiers` adicione **Data Transfer**.
2. Em *Source* escolha o **mesh base** (o que já está devidamente skinned).
3. Em *Data Type* marque **Vertex Groups**; em *Vertex Mapping* escolha **Nearest Face Interpolated** (boa para topo diferentes). Clique em **Generate Data Layers** e depois **Apply**.
4. Agora adicione um modifier **Armature**, em *Object* aponte para o Armature (ou parent com `Ctrl+P → With Empty Groups` e depois remova grupos vazios).
5. Vá em `Pose Mode` do Armature e teste as animações.

> Dica: se precisar ajustar manualmente, entre em `Weight Paint` e pinte/limpe pesos. Use `Normalize` e `Clean` quando for necessário.

### 4 — Ajustes finais no mesh

* `Ctrl + A` → **Apply Rotation & Scale** (faça isso para todas as meshes e para o Armature; escala estranha quebra o bind).
* `Shift + N` para recalcular Normals se necessário.
* Confirme UVs (cada peça tem UVs válidos).
* Materiais: use Principled BSDF com textura em Base Color — o glTF exporta isso bem.

### 5 — Organizar Outliner para facilitar swap in-game

Organize no Outliner com *Empties* ou *Collections*, por exemplo:

```
SLOT_Hair
  Hair_Option_A
  Hair_Option_B
SLOT_Chest
  Armor_Option_A
  Armor_Option_B
```

No export, cada objeto mantém seu nome — isso facilita localizar no Three.js.

### 6 — Animações (várias ações)

Se tiver várias animações (Idle, Run, Jump), transforme cada Action num *NLA track*:

1. Dope Sheet → Action Editor → para cada action clique em **Push Down** (coloca como strip no NLA).
2. Renomeie cada strip com o nome da animação desejada (o exporter do glTF exporta as NLA strips como animações separadas).

### 7 — Exportar para glTF / GLB

`File → Export → glTF 2.0 (.glb/.gltf)`

Configurações recomendadas:

* **Format**: `glb` (binário, tudo embutido)
* **Include**: `Selected Objects` (se você selecionar Armature + meshes) ou `Active Collection`
* **Transform**: (normalmente o padrão do exporter já ajusta, mas confirme que escala é 1 e Up = Y)
* **Geometry**: `Apply Modifiers` ON, `UVs` ON, `Normals` ON, `Tangents` se quiser
* **Skinning**: (exporter incluirá por padrão)
* **Animation**: marcar `Animations`; se usou NLA strips, elas serão exportadas como múltiplas animações

Exporte e abra o `.glb` num visualizador (ex.: [https://gltf-viewer.donmccurdy.com/](https://gltf-viewer.donmccurdy.com/)) para checar.

---

# 3) Workflow recomendado no Three.js (modo mais simples e robusto)

**Carregue o `.glb` único (contendo armature + todas as peças) e troque visibilidade das peças ou clone o modelo usando `SkeletonUtils` para múltiplas instâncias.**

Exemplo (ES modules, Three rXXX):

```js
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { SkeletonUtils } from 'three/examples/jsm/utils/SkeletonUtils.js';

const loader = new GLTFLoader();
const scene = new THREE.Scene();

// carrega o glb único
loader.load('character_modular.glb', (gltf) => {
  const model = gltf.scene;
  scene.add(model);

  // exemplo: procurar um slot e trocar visibilidade
  // supondo que no Blender você tenha colocado opções dentro do group 'SLOT_Hair'
  const slotHair = model.getObjectByName('SLOT_Hair');
  if (slotHair) {
    // esconder todas
    slotHair.children.forEach(c => c.visible = false);
    // ativar uma opção específica
    const opc = slotHair.getObjectByName('Hair_Option_B');
    if (opc) opc.visible = true;
  }

  // clone para criar outro player independente (sem compartilhar pose interna)
  const clone = SkeletonUtils.clone(model);
  clone.position.x = 2;
  scene.add(clone);

  // animações
  const mixer = new THREE.AnimationMixer(model);
  const idle = gltf.animations.find(a => a.name === 'Idle');
  if (idle) mixer.clipAction(idle).play();
});
```

Vantagens:

* **robusto** — não precisa remapear bones no runtime
* performance: um `.glb` com tudo embutido funciona bem (texturas embutidas)

---

# 4) Alternativa avançada — carregar as partes separadas em arquivos diferentes (opção mais complexa)

Se você **obrigatoriamente** precisa carregar hair/roupa em `.glb` separados (download dinâmico), seu *melhor esforço* é:

1. Em Blender, **exporte**:

   * `base.glb`: somente armature + base mesh + animações.
   * `hair_A.glb`: hair **skinned ao mesmo armature** (no Blender você deve ter usado o mesmo armature ao skinnear — o arquivo conterá o armature também).
2. No Three.js: load do `base.glb`, depois load do `hair_A.glb`.
3. Você precisa **remapear** o skinnedMesh do hair para usar os objetos `Bone` do skeleton do base. Isso exige que **os bone names e bindposes sejam idênticos** (mesma hierarquia e rest pose).

Função de exemplo para tentar rebind por nomes (atenção: só funciona se as rest poses forem compatíveis — pode falhar/ter offsets se não forem idênticas):

```js
function rebindSkinnedMeshToBase(skinnedMesh, baseSkinnedMesh) {
  const baseSkeleton = baseSkinnedMesh.skeleton;
  const srcSkeleton = skinnedMesh.skeleton;

  // mapa de nome -> bone do base
  const baseMap = {};
  baseSkeleton.bones.forEach(b => baseMap[b.name] = b);

  // nova lista de bones (na ordem das bones do skinnedMesh)
  const newBones = srcSkeleton.bones.map(b => baseMap[b.name] || b);

  // mapear inverses: procurar bone inverso correspondente do base
  const newInverses = srcSkeleton.boneInverses.map((inv, i) => {
    const name = srcSkeleton.bones[i].name;
    if (baseMap[name]) {
      const idx = baseSkeleton.bones.indexOf(baseMap[name]);
      return baseSkeleton.boneInverses[idx].clone();
    }
    return inv.clone();
  });

  const newSkeleton = new THREE.Skeleton(newBones, newInverses);

  // bind ao novo skeleton — bindMatrix costuma ser o skinnedMesh.bindMatrix salvo no GLTF
  skinnedMesh.bind(newSkeleton, skinnedMesh.bindMatrix);
  skinnedMesh.updateMatrixWorld(true);
}
```

Uso (exemplo):

```js
// assume baseLoaded e partLoaded já prontos
const baseSkinnedMesh = baseScene.getObjectByProperty('type', 'SkinnedMesh');
partScene.traverse(c => {
  if (c.isSkinnedMesh) {
    rebindSkinnedMeshToBase(c, baseSkinnedMesh);
    baseScene.add(c); // adiciona hair ao personagem base
  } else {
    baseScene.add(c);
  }
});
```

**AVISO:** isso é frágil — somente use se:

* as *poses de bind* e *nomes de bones* forem idênticos,
* você estiver confortável em depurar matrices de bind em caso de offset.

Se possível, prefira a abordagem do *único glb exportado do Blender*.

---

# 5) Checklist / Troubleshooting rápido

* [ ] Armature com escala = 1, rotação = 0; aplique transform (`Ctrl+A`).
* [ ] Bone names limpos e únicos.
* [ ] Vertex groups de cada peça correspondem aos bones (verifique no painel `Object Data > Vertex Groups`).
* [ ] Depois de transferir pesos, adicionar `Armature` modifier apontando pro Armature (ordem de modifiers: Armature geralmente acima de outros modifiers que deformam).
* [ ] Se a peça “segue torto”: verifique o `bind pose` (compare a posição dos bones no rest pose vs rest pose do arquivo original).
* [ ] Para múltiplas animações, use NLA strips e renomeie as faixas antes do export.
* [ ] Teste o `.glb` no glTF Viewer antes de integrar ao jogo.
* [ ] No Three.js: para clonar personagens use `SkeletonUtils.clone()` (do pacote examples) para evitar conflitos de skeleton/animation sharing.

---

# 6) Quer eu faça pra você?

Posso:

* te dar **um checklist final** com uma ordem de ações curta pra seguir no Blender;
* ou **um script passo-a-passo** mais curto com os comandos do Blender e nomes exatos dos menus (se preferir em inglês/português);
* ou **um exemplo completo de projeto Three.js** (pequeno app) que carrega um `.glb` modular e troca as partes via UI.

Qual desses você quer que eu gere agora? (ex.: “exemplo Three.js” ou “checklist curto” ou “script detalhado Blender em inglês”)
