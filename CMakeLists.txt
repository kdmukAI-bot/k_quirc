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

# Quad-ROI blend gate - on a failed seed pass with detection anchors, score the
# mid-gray blend contamination inside the detected QR box (always reported via
# the adaptive stats) and, when the runtime gate is enabled via
# k_quirc_set_blend_gate_permille(), bail out of the sweep on blended frames.
# Requires K_QUIRC_ADAPTIVE_THRESHOLD. The runtime gate defaults to disabled
# (0), so compiling this in changes no behavior by itself. Comment out to
# disable.
target_compile_definitions(${COMPONENT_LIB} PUBLIC K_QUIRC_BLEND_GATE)

# Numeric sweep-cap parameter - k_quirc_set_sweep_cap() overrides the effort
# level's built-in pass cap (FAST 4 / THOROUGH full ladder) with an explicit
# per-instance probe budget, so per-board caps can be tuned without rebuilding.
# Default 0 = effort defaults; compiling this in changes no behavior by itself.
# Comment out to disable (the setter stays callable as a no-op).
target_compile_definitions(${COMPONENT_LIB} PUBLIC K_QUIRC_SWEEP_CAP)

# Sweep ladder selection - k_quirc_set_ladder_select() switches between the
# stock ladder and an additive-deep table extending to +/-40 (for THOROUGH
# static-media scans). Raises the threshold-offset clamp to 40 so the deep
# rungs are reachable (the adaptive +/-1 wander stays observable via
# locked_offset). Default 0 = stock ladder; compiling this in changes no
# behavior by itself. Comment out both lines to disable (the setter stays
# callable as a no-op and the clamp reverts to 20).
target_compile_definitions(${COMPONENT_LIB} PUBLIC K_QUIRC_LADDER_SELECT)
target_compile_definitions(${COMPONENT_LIB} PUBLIC K_QUIRC_THRESHOLD_OFFSET_MAX=40)

# FORK-ONLY instrumentation (never an upstream candidate): the seed-override /
# lock-freeze debug API (k_quirc_instr_seed_override) used by the shadow-data
# instrumentation runs. Comment out to disable (APIs stay callable as no-ops).
target_compile_definitions(${COMPONENT_LIB} PUBLIC K_QUIRC_INSTR)

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
