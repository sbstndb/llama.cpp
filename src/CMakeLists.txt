llama_add_compile_flags()

#
# libraries
#

# llama

file(GLOB LLAMA_MODELS_SOURCES "models/*.cpp")

add_library(llama
            ../include/llama.h
            llama.cpp
            llama-adapter.cpp
            llama-arch.cpp
            llama-batch.cpp
            llama-chat.cpp
            llama-context.cpp
            llama-cparams.cpp
            llama-grammar.cpp
            llama-graph.cpp
            llama-hparams.cpp
            llama-impl.cpp
            llama-io.cpp
            llama-kv-cache.cpp
            llama-kv-cache-iswa.cpp
            llama-kv-cache-dsa.cpp
            llama-kv-cache-dsv4.cpp
            llama-memory.cpp
            llama-memory-hybrid.cpp
            llama-memory-hybrid-iswa.cpp
            llama-memory-recurrent.cpp
            llama-mmap.cpp
            llama-model-loader.cpp
            llama-model-saver.cpp
            llama-model.cpp
            llama-quant.cpp
            llama-sampler.cpp
            llama-vocab.cpp
            unicode-data.cpp
            unicode.cpp
            unicode.h
            ${LLAMA_MODELS_SOURCES}
            )

set_target_properties(llama PROPERTIES
    VERSION ${LLAMA_INSTALL_VERSION}
    SOVERSION 0
    MACHO_CURRENT_VERSION 0 # keep macOS linker from seeing oversized version number
)

target_include_directories(llama PRIVATE .)
target_include_directories(llama PUBLIC ../include)
target_compile_features   (llama PRIVATE cxx_std_17) # don't bump

target_link_libraries(llama PUBLIC ggml)

# --- gguf-artifact payload source (validation swap) -------------------------
# Routes tensor payload reads through the gguf-artifact C ABI when GGUFA_LOADER
# is set at runtime. Paths point at the pre-built gguf_artifact C ABI.
set(GGUFA_INCLUDE_DIR "/root/gguf_artifact/include" CACHE PATH "gguf-artifact include dir")
set(GGUFA_LIBRARY    "/root/gguf_artifact/build/libgguf_artifact_c.so" CACHE FILEPATH "gguf-artifact C ABI shared lib")
if(EXISTS "${GGUFA_LIBRARY}")
    target_include_directories(llama PRIVATE "${GGUFA_INCLUDE_DIR}")
    target_link_libraries(llama PRIVATE "${GGUFA_LIBRARY}")
    set_target_properties(llama PROPERTIES BUILD_RPATH "/root/gguf_artifact/build")
else()
    message(WARNING "gguf-artifact C ABI not found at ${GGUFA_LIBRARY}; swap disabled (builds vanilla)")
endif()

if (BUILD_SHARED_LIBS)
    set_target_properties(llama PROPERTIES POSITION_INDEPENDENT_CODE ON)
    target_compile_definitions(llama PRIVATE LLAMA_BUILD)
    target_compile_definitions(llama PUBLIC  LLAMA_SHARED)
endif()
