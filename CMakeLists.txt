set(source_files
    model/orbital-elements.cc
    model/sgp4-mobility-model.cc
    model/ntn-visibility-index.cc
    model/ntn-xnap-messages.cc
    model/ntn-xn-handover.cc
    model/walker-constellation.cc
    model/contact-graph-scheduler.cc
    # 4.4.4 — Contact-graph router integration (Roadmap §4.4 deep work)
    model/contact-graph-router.cc
    # 4.4.11 — TR 38.821 + Starlink calibration corpus (Roadmap §4.4.11)
    model/tr38821-corpus.cc
    # Per-packet C/N0 -> BLER error model for GSL/ISL hops (replaces binary gates).
    model/ntn-sat-link-error-model.cc
)

set(header_files
    model/orbital-elements.h
    model/sgp4-mobility-model.h
    model/ntn-visibility-index.h
    model/ntn-xnap-messages.h
    model/ntn-xn-handover.h
    model/walker-constellation.h
    model/contact-graph-scheduler.h
    # 4.4.4 — Contact-graph router integration (Roadmap §4.4 deep work)
    model/contact-graph-router.h
    # 4.4.11 — TR 38.821 + Starlink calibration corpus (Roadmap §4.4.11)
    model/tr38821-corpus.h
    model/ntn-sat-link-error-model.h
)

set(test_sources
    test/ntn-constellation-test-suite.cc
)

# Optional thz-ntn dependency: provides the ITU-R P.676/P.618 gaseous, rain
# and scintillation models used by NtnSatLinkErrorModel. It exists in the
# ns3-ntn-toolkit tree but not in every tree. Detect via the enabled-contrib
# list (thz-ntn configures after this module alphabetically, so a TARGET
# check here would always miss) and only then link it and define
# NTN_CONSTELLATION_HAS_THZ_NTN so the atmospheric chain compiles in.
# Without it the model builds FSPL-only and reports zero excess loss.
set(ntn_constellation_optional_libs "")
if("thz-ntn" IN_LIST contrib_libs_to_build)
  set(ntn_constellation_optional_libs ${libthz-ntn})
endif()

build_lib(
  LIBNAME ntn-constellation
  SOURCE_FILES ${source_files}
  HEADER_FILES ${header_files}
  LIBRARIES_TO_LINK
    ${libcore}
    ${libmobility}
    # Full Vallado SGP4 backend reuses the satellite module's reference sgp4unit.
    ${libsatellite}
    ${ntn_constellation_optional_libs}
  TEST_SOURCES ${test_sources}
)

if(ntn_constellation_optional_libs)
  target_compile_definitions(${libntn-constellation} PUBLIC NTN_CONSTELLATION_HAS_THZ_NTN)
endif()
