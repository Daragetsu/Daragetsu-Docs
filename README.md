## Daragetsu's Docs on Minecraft Forge

this doc is just a documentation on stuff I wanna keep track of, a lot of bugs and 'how to's I've found,

*this doc is based on Forge 1.20*

### - Feature Order Cycle
 > when creating an custom biome or adding an vanilla placed feature to an already existing biome, sometimes it will throw an "Feature Order Cycle" error when ran with other mods that add biomes, it happens because those biomes also added the vanilla placed feature but in a different order than yours, I have found this solution to be the simplest and also written in the Neoforged docs
  - Make a new configured feature json in your own namespace(e.g. `resources/data/examplemod/worldgen/configured_feature/grass_patch.json`) and copy over the contents of the vanilla configured feature which you want to add,
  - Make a new placed feature json in your own namespace(e.g. `resources/data/examplemod/worldgen/placed_feature/grass_patch.json`) and copy over the contents of the vanilla placed feature which you want to add, and refrence your configured feature instead of the vanilla one,
  - add the placed feature in the biome instead of the vanilla placed feature,

---

### - Making Structures Spawn Into The Ground(e.g. Craters)
 > this is for jigsaw structures
  - in the template pool json(e.g. `resources/data/examplemod/worldgen/template_pool`) in the element which you want to spawn into the ground, add an gravity processor like:
  - ```
    "elements": [
        {
            "weight": 1,
            "element": {
                "location": "examplemod:crater",
                "processors": [
                {
                    "processor_type": "minecraft:gravity",
                    "heightmap": "WORLD_SURFACE",
                    "offset": -4
                }
                ],
                "projection": "rigid",
                "element_type": "minecraft:single_pool_element"
            }
        }
    ]

  - this will make the structure spawn 4 blocks below where it would have originally spawned, you can change the `heightmap` to be whichever heightmap you're using for your structure,

---

### - Making Blocks(e.g. stairs, slabs, walls) in Structues that spawn inside of water not waterlog(e.g. Ships)
 > this is for jigsaw structures
  - create a custom structure processor and register it like:
  - Processor Class: 
  - ```
        public class WarshipProcessor extends StructureProcessor{
            public static final Codec<WarshipProcessor> CODEC = Codec.unit(WarshipProcessor::new);
            public WarshipProcessor(){
            }
            @Override
            public StructureBlockInfo process(LevelReader level, BlockPos pos, BlockPos pos2, StructureBlockInfo sbi, StructureBlockInfo sbi2, StructurePlaceSettings settings, StructureTemplate template) {
                return super.process(level, pos, pos2, sbi, sbi2, settings.setKeepLiquids(false), template);
            }
            @Override
            protected StructureProcessorType<?> getType() {
                return ModStructureProcessors.WARSHIP_PROCESSOR.get();
            }
        }
        - what this does is make the structure not keep lequids from where it's generating,

  - ModStructureProcessors: 
  - ```
        public class ModStructureProcessors {
            public static final DeferredRegister<StructureProcessorType<?>> STRUCTURE_PROCESSOR_TYPES = DeferredRegister
                    .create(Registries.STRUCTURE_PROCESSOR, ExampleMod.MOD_ID);

            public static final RegistryObject<StructureProcessorType<WarshipProcessor>> WARSHIP_PROCESSOR = STRUCTURE_PROCESSOR_TYPES.register("warship_processor", ()->()->WarshipProcessor.CODEC);

            public static void register(IEventBus modEventBus) {
                STRUCTURE_PROCESSOR_TYPES.register(modEventBus);
            }
        }

  - register it in your Main class `ModStructureProcessors.register(modEventBus);`,

  - make a new json file under `resources/data/examplemod/worldgen/processor_list` with:
  - ```
        {
            "processors": [
                {
                "processor_type": "examplemod:warship_processor"
                }
            ]
        }

  - and then add it to your template pool:
  - ``` 
    "elements": [
        {
            "weight": 1,
            "element": {
                "element_type": "minecraft:single_pool_element",
                "projection": "rigid",
                "location": "examplemod:ship",
                "processors": "examplemod:warship_processor"
            }
        }
    ]

---

### - Adding Glowmasks with geckolib
 > you need a custom renderer for this
 - Add an AutoGlowingLayer in your renderer constructor:
 - ``` 
    this.addRenderLayer(new AutoGlowingGeoLayer<>(this){
            @Override
            public void render(PoseStack poseStack, T animatable, BakedGeoModel bakedModel, RenderType renderType, MultiBufferSource bufferSource, VertexConsumer buffer, float partialTick, int packedLight, int packedOverlay) {
                RenderType emissiveRenderType = getRenderType(animatable);
		        getRenderer().reRender(
                    bakedModel,
                    poseStack,
                    bufferSource,
                    animatable,
                    emissiveRenderType,
					bufferSource.getBuffer(emissiveRenderType),
                    partialTick,
                    LightTexture.FULL_BRIGHT,
                    OverlayTexture.NO_OVERLAY,
                    1,
                    1,
                    1,
                    1
                );
            }
        })

 - copy your entity texture and name it `<texture_name>_glowmask.png`, open the new file and remove any pixels which you do not want glowing(it will leave you with an mostly empty image)

---

### - MORE WILL BE ADDED AS I REMEMBER THEM