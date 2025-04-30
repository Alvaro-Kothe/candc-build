cmake_minimum_required(VERSION 3.25)
project(CandC LANGUAGES C CXX)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -W -Wall -Wextra -fPIC -O3")

include_directories(src/include)

# Platform-specific
if(APPLE)
  # macOS-specific settings
  set(PLAT_PORT_SRC src/lib/ports/bsd.cc src/lib/ports/unix_common.cc
                    src/lib/ports/colour.cc)
  # (Optionally add macOS-specific sources)
elseif(WIN32)
  # Windows ports
  set(PLAT_PORT_SRC src/lib/ports/mingw.cc src/lib/ports/mono.cc)
else()
  # assume Unix/Linux
  set(PLAT_PORT_SRC src/lib/ports/unix.cc src/lib/ports/unix_common.cc
                    src/lib/ports/colour.cc)
endif()

# SWI-Prolog to build boxer
find_package(SWIPL REQUIRED)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set(BASE_SRCS
    src/lib/base.cc
    ${PLAT_PORT_SRC}
    src/lib/timer.cc
    src/lib/input.cc
    src/lib/word.cc
    src/lib/version.cc
    src/lib/licence.cc
    src/lib/tagset.cc
    src/lib/lexicon.cc)

set(CONFIG_SRCS
    src/lib/utils/escape.cc src/lib/config/node.cc src/lib/config/option.cc
    src/lib/config/config.cc src/lib/io/format.cc)

set(IO_SRCS
    src/lib/io/stream.cc
    src/lib/io/reader.cc
    src/lib/io/reader_group.cc
    src/lib/io/reader_horiz.cc
    src/lib/io/reader_multi_horiz.cc
    src/lib/io/reader_factory.cc
    src/lib/io/writer.cc
    src/lib/io/writer_stream.cc
    src/lib/io/writer_group.cc
    src/lib/io/writer_horiz.cc
    src/lib/io/writer_multi_horiz.cc
    src/lib/io/writer_vert.cc
    src/lib/io/writer_multi_vert.cc
    src/lib/io/writer_format.cc
    src/lib/io/writer_factory.cc)

set(TAG_SRCS
    src/lib/tagger/tagdict.cc src/lib/tagger/tagsetdict.cc
    src/lib/tagger/_baseimpl.cc src/lib/tagger/tagger.cc
    src/lib/tagger/taghist.cc)

set(FEATURE_SRCS src/lib/gazetteers.cc src/lib/tagger/taghist.cc
                 src/lib/tagger/unigram.cc)

set(MODEL_SRCS
    src/lib/model/types.cc
    src/lib/model/registry.cc
    src/lib/model/affix.cc
    src/lib/model/unigram.cc
    src/lib/model/bigram.cc
    src/lib/model/trigram.cc
    src/lib/model/model.cc
    src/lib/model/cache.cc)

set(TAGGER_SRCS ${BASE_SRCS} ${CONFIG_SRCS} ${IO_SRCS} ${TAG_SRCS}
                ${MODEL_SRCS} ${FEATURE_SRCS})

set(CCG_SRCS
    src/lib/parser/atom.cc
    src/lib/parser/feature.cc
    src/lib/parser/varid.cc
    src/lib/parser/category.cc
    src/lib/parser/markedup.cc
    src/lib/parser/categories.cc
    src/lib/parser/gr.cc
    src/lib/parser/gr_constraints.cc
    src/lib/parser/relations.cc
    src/lib/parser/supercat.cc
    src/lib/parser/equiv.cc
    src/lib/parser/chart.cc
    src/lib/parser/unify.cc
    src/lib/parser/treebank.cc
    src/lib/parser/variable.cc
    src/lib/parser/rule.cc
    src/lib/tree/catvalue.cc
    src/lib/tree/dependency.cc
    src/lib/tree/distance.cc
    src/lib/tree/rule.cc
    src/lib/tree/depdist.cc
    src/lib/tree/genrule.cc
    src/lib/lexicon.cc
    src/lib/parser/parser.cc
    src/lib/parser/depscore.cc
    src/lib/parser/canonical.cc
    src/lib/parser/generator.cc
    src/lib/parser/constraint.cc
    src/lib/parser/print_deps.cc
    src/lib/parser/print_prolog.cc
    src/lib/parser/print_boxer.cc
    src/lib/parser/print_ccgbank.cc
    src/lib/parser/print_grs.cc
    src/lib/parser/print_xml.cc
    src/lib/parser/print_debug.cc
    src/lib/parser/print_js.cc
    src/lib/parser/print_latex.cc
    src/lib/parser/print_factory.cc
    src/lib/parser/decoder.cc
    src/lib/parser/decoder_derivs.cc
    src/lib/parser/decoder_derivs_random.cc
    src/lib/parser/decoder_deps_recall.cc
    src/lib/parser/decoder_factory.cc
    src/lib/parser/beam.cc
    src/lib/parser/count_rules.cc
    src/lib/parser/print_forest.cc
    src/lib/parser/inside_outside.cc
    src/lib/parser/feature_root.cc
    src/lib/parser/feature_rule.cc
    src/lib/parser/feature_rule_head.cc
    src/lib/parser/feature_dep.cc
    src/lib/parser/feature_dep_dist.cc
    src/lib/parser/feature_rule_dep.cc
    src/lib/parser/feature_rule_dep_dist.cc
    src/lib/parser/feature_genrule.cc
    src/lib/parser/integration.cc
    src/lib/parser/rule_instances.cc)

set(EXTRA_SRCS
    src/lib/tagger/pos.cc src/lib/tagger/chunk.cc src/lib/tagger/ner.cc
    src/lib/tagger/super.cc src/lib/relations/morpha.c
    src/lib/cluster_no_mpi.cc)

# Main entry
set(CANDC_MAIN src/main/candc.cc src/lib/candc.cc)

# Full target sources
set(CANDC_SOURCES ${TAGGER_SRCS} ${CCG_SRCS} ${EXTRA_SRCS} ${CANDC_MAIN})

# build candc
add_executable(candc ${CANDC_SOURCES})

file(GLOB_RECURSE BOXER_SOURCES src/prolog/boxer/*.pl src/prolog/lib/*.pl)

set(BOXER_OUTPUT ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/boxer)

add_custom_command(
  OUTPUT ${BOXER_OUTPUT}
  COMMAND
    ${SWIPL_EXECUTABLE} -g
    "['src/prolog/boxer/boxer'], qsave_program('${BOXER_OUTPUT}',[goal=start,stand_alone=true,foreign=save]), halt."
  VERBATIM
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  DEPENDS ${BOXER_SOURCES}
  COMMENT "Compiling Prolog sources into boxer")

add_custom_target(build_boxer ALL DEPENDS ${BOXER_OUTPUT})
