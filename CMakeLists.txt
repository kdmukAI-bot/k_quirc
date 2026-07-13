idf_component_register(
    SRCS
        "src/k_quirc.c"
        "src/k_quirc_version.c"
        "src/k_quirc_identify.c"
        "src/k_quirc_decode.c"
    INCLUDE_DIRS "include"
    PRIV_INCLUDE_DIRS "src"
)

# Adaptive threshold control loop - adjusts threshold based on timing pattern analysis
# Comment out to disable
target_compile_definitions(${COMPONENT_LIB} PUBLIC K_QUIRC_ADAPTIVE_THRESHOLD)

# Bilinear threshold interpolation - uses four quadrants for uneven lighting
# Comment out to disable (uses single global threshold instead)
target_compile_definitions(${COMPONENT_LIB} PUBLIC K_QUIRC_BILINEAR_THRESHOLD)

# Local (Bradley) adaptive threshold - a finer per-window binarization used ONLY
# as the failure-gated second pass in k_quirc_decode_adaptive (THOROUGH effort).
# Zero cost unless a frame fails the entire global offset sweep. Comment out to
# disable (the adaptive decode then relies on the global sweep alone).
target_compile_definitions(${COMPONENT_LIB} PUBLIC K_QUIRC_LOCAL_THRESHOLD)

# Uncomment to enable the inverted-colors retry pass when no QR is found.
# target_compile_definitions(${COMPONENT_LIB} PUBLIC K_QUIRC_INVERTED_RETRY)

# Route decoder context + flood-fill scratch to PSRAM - Kconfig-driven under ESP-IDF
if(CONFIG_K_QUIRC_SCRATCH_IN_PSRAM)
    target_compile_definitions(${COMPONENT_LIB} PUBLIC K_QUIRC_SCRATCH_IN_PSRAM)
endif()

# Debug visualization - Kconfig-driven when building under ESP-IDF
if(CONFIG_K_QUIRC_DEBUG)
    target_compile_definitions(${COMPONENT_LIB} PUBLIC K_QUIRC_DEBUG)
endif()
