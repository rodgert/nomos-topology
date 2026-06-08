# nomos-topology

Shared data schema for the [nomos-studio](https://github.com/nomos-studio) synthesis
topology layer.  Defines validated Clojure maps describing kairos-grid patches,
kairos session graphs, nomos-rt modulation bindings, and the control tree that
wires the nous compositional surface to hardware.

Used as a dependency by [nous](https://github.com/nomos-studio/nous) and
[alembic](https://github.com/nomos-studio/alembic); consumed as a header-only
C++ constant library by kairos and kairos-grid.

## Clojure

```clojure
;; project.clj
[nomos-topology "0.1.0"]
```

```clojure
(require '[nomos.topology.core :as topo])

;; Build a session topology map
(def my-session
  (topo/session
    {:nodes [{:id    :voice
              :type  :kairos-grid
              :patch {:modules [{:type "plaits" :params {:harmonics 0.5}}
                                {:type "audio-out"}]
                      :cables  [[0 0 1 0] [0 1 1 1]]}}]
     :routes [{:from [:voice 0] :to :master/left}
              {:from [:voice 1] :to :master/right}]
     :control-tree
     {:controls {:pitch {:target [:voice "plaits/note"] :range [0 127] :type :midi-note}}}}))

;; Validate against the Malli schema
(topo/valid? my-session)   ;=> true
(topo/explain my-session)  ;=> nil on success, error map on failure
```

## C++

The C++ side is header-only: EDN keyword constants for all topology keys,
usable without Clojure at the kairos / kairos-grid layer.

```cmake
# CMakeLists.txt — via FetchContent or local path
FetchContent_Declare(nomos_topology
    GIT_REPOSITORY https://github.com/nomos-studio/nomos-topology.git
    GIT_TAG        0.1.0)
FetchContent_MakeAvailable(nomos_topology)

target_link_libraries(my-target PRIVATE nomos::topology)
```

```cpp
#include <nomos/topology/keys.hpp>

// EDN keyword constants for session graph keys
nomos::topology::k_nodes   // :nodes
nomos::topology::k_routes  // :routes
nomos::topology::k_patch   // :patch
```

## Schema overview

| Concept | Description |
|---|---|
| `Session` | Top-level map: `:topology` + `:control-tree` |
| `Topology` | `:nodes` (plugin graph) + `:routes` (audio connections) + `:modulations` |
| `Node` | `{:id kw :type kw-or-str :patch ...}` — a plugin instance in the graph |
| `Route` | `{:from endpoint :to endpoint}` — an audio connection between nodes |
| `Patch` | `{:modules [...] :cables [...]}` — kairos-grid internal module graph |
| `ControlTree` | `{:controls {name {:target [...] :range [...] :cc N}}}` |

## Running tests

```bash
lein test
```

12 tests, 50 assertions.

## License

EPL-2.0 — see [LICENSE](LICENSE).
