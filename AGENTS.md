# SketchOpn Custom OpenCascade.js Build Agent

## Goal
- Produce a custom `opencascade.js` build that carries SketchOpn-specific bindings.
- Offload the build to dependable x86_64 infrastructure because Apple Silicon hosts fail at runtime and Rosetta/qemu cannot emulate the full toolchain.

## Reference: OpenCascade.js Custom Build Flow
- Custom builds live in YAML files that define one or more builds (`mainBuild`, optional extras) plus optional shared snippets like `additionalCppCode` at the top level.
- Pull the official toolchain with `docker pull donalffons/opencascade.js`.
- Author the YAML with your bindings and `emccFlags`. If you add helper C++ code, place it as `additionalCppCode` alongside `mainBuild`, not nested inside the build definition.
- Run the container from the directory that holds the YAML:
  ```bash
  docker run --rm -it \
    -v "$(pwd):/src" \
    -u "$(id -u):$(id -g)" \
    donalffons/opencascade.js \
    your-build.yml
  ```
- The build emits `{name}.js`, `{name}.wasm`, and `{name}.d.ts` for import via `initOpenCascade`.

## Build It on x86_64
- Keep the build inside GitHub Actions’ `ubuntu-latest` (x86_64) runners to dodge Apple Silicon crashes.
- The container is multi-arch capable; add `--platform linux/amd64` so the job never falls back to arm64.

## Concrete Workflow for SketchOpn

### 1. Fork and prepare the repo
- Fork https://github.com/donalffons/opencascade.js into your account/org.
- Clone locally:
  ```bash
  git clone git@github.com:<you>/opencascade.js.git
  cd opencascade.js
  ```
- Add `builds/sketchopn.custom.yml` with the SketchOpn bindings, toolchain flags, and helper C++ (top-level `additionalCppCode`). You can adapt the version we keep under `ops/opencascade/`:
  ```yaml
mainBuild:
  name: occt.sketchopn.js
  bindings:
    - symbol: RWGltf_CafWriter
    - symbol: Standard_Transient
    - symbol: Standard_Failure
    - symbol: TCollection_AsciiString
    - symbol: Handle_TDocStd_Document
    - symbol: TDocStd_Document
    - symbol: CDM_Document
    - symbol: TColStd_IndexedDataMapOfStringString
    - symbol: Message_ProgressRange
    - symbol: TCollection_ExtendedString
    - symbol: XCAFDoc_DocumentTool
    - symbol: TDataStd_GenericEmpty
    - symbol: TDF_Attribute
    - symbol: XCAFDoc_ShapeTool
    - symbol: Handle_XCAFDoc_ShapeTool
    - symbol: XCAFDoc_VisMaterialTool
    - symbol: Handle_XCAFDoc_VisMaterialTool
    - symbol: XCAFDoc_VisMaterial
    - symbol: Handle_XCAFDoc_VisMaterial
    - symbol: XCAFDoc_VisMaterialPBR
    - symbol: Quantity_Color
    - symbol: Quantity_ColorRGBA
    - symbol: BRepMesh_IncrementalMesh
    - symbol: BRepMesh_DiscretRoot
    - symbol: BRepPrimAPI_MakeOneAxis
    - symbol: BRepPrimAPI_MakeSweep
    - symbol: BRepPrimAPI_MakeBox
    - symbol: BRepPrimAPI_MakeCylinder
    - symbol: BRepPrimAPI_MakePrism
    - symbol: BRepAlgoAPI_BooleanOperation
    - symbol: BRepAlgoAPI_Algo
    - symbol: BRepAlgoAPI_BuilderAlgo
    - symbol: BRepAlgoAPI_Fuse
    - symbol: BRepAlgoAPI_Cut
    - symbol: BRepAlgoAPI_Common
    - symbol: BRepAlgoAPI_Check
    - symbol: BRepBuilderAPI_MakeShape
    - symbol: BRepBuilderAPI_Command
    - symbol: BRepBuilderAPI_MakeEdge
    - symbol: BRepBuilderAPI_MakeWire
    - symbol: BRepBuilderAPI_MakeFace
    - symbol: BRepBuilderAPI_MakePolygon
    - symbol: BRepBuilderAPI_Transform
    - symbol: BRepBuilderAPI_ModifyShape
    - symbol: BRepBuilderAPI_MakeSolid
    - symbol: BRepFilletAPI_MakeFillet
    - symbol: BRepFilletAPI_LocalOperation
    - symbol: BRepOffsetAPI_MakeThickSolid
    - symbol: BRepOffsetAPI_MakeOffsetShape
    - symbol: BRepOffsetAPI_ThruSections
    - symbol: BRepOffset_Mode
    - symbol: BOPAlgo_Options
    - symbol: BRep_Tool
    - symbol: BRepTools
    - symbol: BRepAdaptor_Surface
    - symbol: Adaptor3d_Surface
    - symbol: LocOpe_Prism
    - symbol: GeomAbs_SurfaceType
    - symbol: GeomAdaptor_Surface
    - symbol: Geom_Surface
    - symbol: Geom_Geometry
    - symbol: Handle_Geom_Surface
    - symbol: Geom_Plane
    - symbol: Handle_Geom_Plane
    - symbol: Geom_ElementarySurface
    - symbol: Geom_Curve
    - symbol: Geom_BoundedCurve
    - symbol: Geom2d_Curve
    - symbol: Geom2d_BoundedCurve
    - symbol: Geom2d_TrimmedCurve
    - symbol: Geom2d_Geometry
    - symbol: Handle_Geom2d_Curve
    - symbol: Handle_Geom2d_TrimmedCurve
    - symbol: GC_MakeSegment
    - symbol: GC_MakeArcOfCircle
    - symbol: GC_Root
    - symbol: GCE2d_MakeSegment
    - symbol: GCE2d_Root
    - symbol: gp
    - symbol: gp_Pnt
    - symbol: gp_Vec
    - symbol: gp_Trsf
    - symbol: gp_Dir
    - symbol: gp_Ax1
    - symbol: gp_Ax2
    - symbol: gp_Ax3
    - symbol: gp_Pnt2d
    - symbol: gp_Dir2d
    - symbol: gp_Ax2d
    - symbol: gp_Pln
    - symbol: Geom2d_Ellipse
    - symbol: Geom2d_Conic
    - symbol: TopoDS
    - symbol: TopoDS_Shape
    - symbol: TopoDS_Face
    - symbol: TopoDS_Edge
    - symbol: TopoDS_Wire
    - symbol: TopoDS_Compound
    - symbol: TopoDS_Vertex
    - symbol: TopoDS_Builder
    - symbol: TopoDS_Iterator
    - symbol: TopAbs_ShapeEnum
    - symbol: TopAbs_Orientation
    - symbol: TopExp
    - symbol: TopExp_Explorer
    - symbol: TopTools_ListOfShape
    - symbol: NCollection_BaseList
    - symbol: NCollection_BaseAllocator
    - symbol: NCollection_BaseMap
    - symbol: TopTools_IndexedMapOfShape
    - symbol: TopLoc_Location
    - symbol: Poly_Triangulation
    - symbol: Handle_Poly_Triangulation
    - symbol: Poly_Array1OfTriangle
    - symbol: Poly_ArrayOfNodes
    - symbol: Poly_Triangle
    - symbol: Poly_PolygonOnTriangulation
    - symbol: Handle_Poly_PolygonOnTriangulation
    - symbol: Poly_Connect
    - symbol: TColStd_Array1OfInteger
    - symbol: TColStd_Array1OfReal
    - symbol: TColgp_Array1OfPnt
    - symbol: IMeshTools_Context
    - symbol: IMeshData_Shape
    - symbol: ChFi3d_FilletShape
    - symbol: GeomAbs_JoinType
    - symbol: GeomAbs_Shape
    - symbol: STEPCAFControl_Reader
    - symbol: STEPCAFControl_Writer
    - symbol: STEPControl_Reader
    - symbol: STEPControl_Writer
    - symbol: IGESControl_Reader
    - symbol: IGESControl_Writer
    - symbol: XSControl_Reader
    - symbol: XSControl_WorkSession
    - symbol: IFSelect_WorkSession
  emccFlags:
    - -flto
    - -fexceptions
    - -sDISABLE_EXCEPTION_CATCHING=0
    - -O3
    - -sEXPORT_ES6=1
    - -sUSE_ES6_IMPORT_META=0
    - -sEXPORTED_RUNTIME_METHODS=['FS']
    - -sINITIAL_MEMORY=200MB
    - -sMAXIMUM_MEMORY=4GB
    - -sALLOW_MEMORY_GROWTH=1
    - -sUSE_FREETYPE=1
    - -sLLD_REPORT_UNDEFINED
    - --no-entry
    - -sENVIRONMENT='web,worker'
additionalCppCode: |
  typedef Handle(IMeshTools_Context) Handle_IMeshTools_Context;
  class OCJS {
  public:
    static Standard_Failure* getStandard_FailureData(intptr_t exceptionPtr) {
      return reinterpret_cast<Standard_Failure*>(exceptionPtr);
    }
  };

  ```

### 2. Add the GitHub Actions workflow
Create `.github/workflows/sketchopn-custom.yml`:
```yaml
name: build-sketchopn-custom

on:
  workflow_dispatch:
  push:
    branches: [ main ]
    paths:
      - 'builds/sketchopn.custom.yml'
      - '.github/workflows/sketchopn-custom.yml'

jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 180

    steps:
      - name: Checkout fork
        uses: actions/checkout@v4

      - name: Pull official ocjs image
        run: docker pull donalffons/opencascade.js

      - name: Build custom wasm
        env:
          YAML_FILE: builds/sketchopn.custom.yml
        run: |
          docker run --rm -i \
            --platform linux/amd64 \
            -v "$PWD:/src" \
            -u "$(id -u):$(id -g)" \
            donalffons/opencascade.js \
            "$YAML_FILE"

      - name: Collect artifacts
        run: |
          mkdir -p dist
          mv occt.sketchopn.js dist/
          mv occt.sketchopn.wasm dist/
          mv occt.sketchopn.d.ts dist/

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: ocjs-sketchopn
          path: dist/**
          retention-days: 14
```

### 3. Run it and fetch the bundle
- Open your fork’s **Actions** tab, pick `build-sketchopn-custom`, and trigger **Run workflow**.
- Once the job finishes, download `ocjs-sketchopn.zip` from the run’s Artifacts panel.
- Extract `occt.sketchopn.{js, wasm, d.ts}` into the SketchOpn repository (e.g., `packages/kernel/public/opencascade/`) and update the worker loader to point at them.

### 4. Optional niceties
- Add a post-build step to publish to Releases or S3 if you want automatic distribution.
- Schedule a nightly build with `on: schedule` to keep artifacts fresh.
- Any push touching the bindings file re-triggers the workflow, so updates stay in sync.

## Modules Added Beyond the Stock Distribution
- **Solid modeling & push/pull**: Prism builders (`BRepPrimAPI_MakePrism`, `LocOpe_Prism`), boolean ops (`BRepAlgoAPI_*`), fillets (`BRepFilletAPI_*`), offsets (`BRepOffsetAPI_*`), and helpers (gc/gce2d, `gp_*`, `Geom*`).
- **Meshing & tessellation**: `BRepMesh_*`, `IMeshTools_Context`, `IMeshData_Shape`, Poly tessellation helpers (`Poly_Array1OfTriangle`, `Poly_ArrayOfNodes`, `Poly_Triangle`, `Poly_Connect`), topology maps (`TopExp`, `TopTools_*`, `NCollection_BaseList`, `NCollection_BaseAllocator`, `NCollection_BaseMap`, `TopLoc_Location`), `TopoDS` family, and color carriers (`Quantity_Color*`).
- **XCAF metadata & materials**: `XCAFDoc_*` tools, `TDocStd_*`, `TDF_*`, `TCollection_*`, plus `Standard_Failure` to surface exception payloads via the helper in `additionalCppCode`.
- **STEP/IGES I/O**: `STEPCAFControl_*`, `STEPControl_*`, `IGESControl_*`, `XSControl_*`, `IFSelect_WorkSession`, plus `Message_ProgressRange` for progress integration.

These bindings are the minimum set that keeps SketchOpn’s pipeline functional; anything outside this list can stay on the default OpenCascade.js build without breaking our features.
