cmake_minimum_required(VERSION 3.0)

project(libpostal)

option(DOWNLOAD_DATA "Download data files" ON)
option(HAVE_SHUF "Have shuf" ON)
option(HAVE_DIRENT_H "Have dirent.h" ON)

set(LIBPOSTAL_DATA_DIR ${CMAKE_CURRENT_LIST_DIR}/data CACHE STRING "Data directory")
set(LIBPOSTAL_REPO_NAME "openvenues/libpostal" CACHE STRING "libpostal repo name")

if(DOWNLOAD_DATA)

    set(LIBPOSTAL_DATA_DIR_VERSION_STRING "v1")

    set(LIBPOSTAL_DATA_FILE_LATEST_VERSION "v1.0.0")
    set(LIBPOSTAL_PARSER_MODEL_LATEST_VERSION "v1.0.0")
    set(LIBPOSTAL_LANG_CLASS_MODEL_LATEST_VERSION "v1.0.0")

    set(LIBPOSTAL_DATA_FILE "libpostal_data.tar.gz")
    set(LIBPOSTAL_PARSER_FILE "parser.tar.gz")
    set(LIBPOSTAL_LANG_CLASS_FILE "language_classifier.tar.gz")

    set(LIBPOSTAL_DATA_FILE_VERSION_PATH ${LIBPOSTAL_DATA_DIR}/base_data_file_version)
    set(LIBPOSTAL_PARSER_MODEL_VERSION_PATH ${LIBPOSTAL_DATA_DIR}/parser_model_file_version)
    set(LIBPOSTAL_LANG_CLASS_MODEL_VERSION_PATH ${LIBPOSTAL_DATA_DIR}/language_classifier_model_file_version)

    set(LIBPOSTAL_BASE_URL "https://github.com/${LIBPOSTAL_REPO_NAME}/releases/download")

    # Download the base data file
    set(LIBPOSTAL_DATA_FILE_URL "${LIBPOSTAL_BASE_URL}/${LIBPOSTAL_DATA_FILE_LATEST_VERSION}/${LIBPOSTAL_DATA_FILE}")
    set(LIBPOSTAL_DATA_FILE_LOCAL_PATH "${LIBPOSTAL_DATA_DIR}/${LIBPOSTAL_DATA_FILE}")
    if(EXISTS ${LIBPOSTAL_DATA_FILE_LOCAL_PATH})
        message(STATUS "Data file already exists")
    else()
        message(STATUS "Downloading data file from ${LIBPOSTAL_DATA_FILE_URL}")
        file(DOWNLOAD ${LIBPOSTAL_DATA_FILE_URL} ${LIBPOSTAL_DATA_FILE_LOCAL_PATH} SHOW_PROGRESS)

        message(STATUS "Extracting data file")
        execute_process(
                COMMAND ${CMAKE_COMMAND} -E tar xzf ${LIBPOSTAL_DATA_FILE_LOCAL_PATH}
                WORKING_DIRECTORY ${LIBPOSTAL_DATA_DIR}
        )
    endif ()

    # Download the parser model
    set(LIBPOSTAL_PARSER_MODEL_URL "${LIBPOSTAL_BASE_URL}/${LIBPOSTAL_PARSER_MODEL_LATEST_VERSION}/${LIBPOSTAL_PARSER_FILE}")
    set(LIBPOSTAL_PARSER_LOCAL_PATH "${LIBPOSTAL_DATA_DIR}/${LIBPOSTAL_PARSER_FILE}")
    if(EXISTS ${LIBPOSTAL_PARSER_LOCAL_PATH})
        message(STATUS "Parser model already exists")
    else()
        message(STATUS "Downloading parser model from ${LIBPOSTAL_PARSER_MODEL_URL}")
        file(DOWNLOAD ${LIBPOSTAL_PARSER_MODEL_URL} ${LIBPOSTAL_PARSER_LOCAL_PATH} SHOW_PROGRESS)

        message(STATUS "Extracting parser model")
        execute_process(
                COMMAND ${CMAKE_COMMAND} -E tar xzf ${LIBPOSTAL_PARSER_LOCAL_PATH}
                WORKING_DIRECTORY ${LIBPOSTAL_DATA_DIR}
        )
    endif()

    # Download the language classifier model
    set(LANG_CLASS_MODEL_URL "${LIBPOSTAL_BASE_URL}/${LIBPOSTAL_LANG_CLASS_MODEL_LATEST_VERSION}/${LIBPOSTAL_LANG_CLASS_FILE}")
    set(LANG_CLASS_LOCAL_PATH "${LIBPOSTAL_DATA_DIR}/${LIBPOSTAL_LANG_CLASS_FILE}")
    if(EXISTS ${LANG_CLASS_LOCAL_PATH})
        message(STATUS "Language classifier model already exists")
    else()
        message(STATUS "Downloading language classifier model from ${LANG_CLASS_MODEL_URL}")
        file(DOWNLOAD ${LANG_CLASS_MODEL_URL} ${LANG_CLASS_LOCAL_PATH} SHOW_PROGRESS)

        message(STATUS "Extracting language classifier model")
        execute_process(
                COMMAND ${CMAKE_COMMAND} -E tar xzf ${LANG_CLASS_LOCAL_PATH}
                WORKING_DIRECTORY ${LIBPOSTAL_DATA_DIR}
        )
    endif()

endif ()

#download_release $LIBPOSTAL_DATA_FILE_VERSION_PATH $LIBPOSTAL_DATA_FILE_LATEST_VERSION $LIBPOSTAL_DATA_DIR $LIBPOSTAL_DATA_FILE_CHUNKS $LIBPOSTAL_DATA_FILE "data file" $BASIC_MODULE_DIRS

set(LIBPOSTAL_LA_SOURCES
        src/strndup.c
        src/libpostal.c
        src/expand.c
        src/address_dictionary.c
        src/transliterate.c
        src/tokens.c
        src/trie.c
        src/trie_search.c
        src/trie_utils.c
        src/string_utils.c
        src/file_utils.c
        src/utf8proc/utf8proc.c
        src/normalize.c
        src/numex.c
        src/features.c
        src/unicode_scripts.c
        src/address_parser.c
        src/address_parser_io.c
        src/averaged_perceptron.c
        src/crf.c
        src/crf_context.c
        src/sparse_matrix.c
        src/averaged_perceptron_tagger.c
        src/graph.c
        src/graph_builder.c
        src/language_classifier.c
        src/language_features.c
        src/logistic_regression.c
        src/logistic.c
        src/minibatch.c
        src/float_utils.c
        src/ngrams.c
        src/place.c
        src/near_dupe.c
        src/double_metaphone.c
        src/geohash/geohash.c
        src/dedupe.c
        src/string_similarity.c
        src/acronyms.c
        src/soft_tfidf.c
        src/jaccard.c
)

add_library(libpostal_la STATIC ${LIBPOSTAL_LA_SOURCES})
target_include_directories(libpostal_la PUBLIC src)

target_compile_definitions(libpostal_la PUBLIC DEFAULT_TRANSLITERATION_PATH="")
target_compile_definitions(libpostal_la PUBLIC DEFAULT_ADDRESS_EXPANSION_PATH="")
target_compile_definitions(libpostal_la PUBLIC LIBPOSTAL_DATA_DIR="${LIBPOSTAL_DATA_DIR}")

target_compile_definitions(libpostal_la PUBLIC HAVE_DIRENT_H=$<BOOL:${HAVE_DIRENT_H}>) # todo: check if this is needed

set(LIBSCANNER_SOURCES src/klib/drand48.c src/scanner.c)
add_library(libscanner STATIC ${LIBSCANNER_SOURCES})
target_include_directories(libscanner PUBLIC src)

# Add libscanner to libpostal_la
target_link_libraries(libpostal_la libscanner)

set(LIBPOSTAL_SOURCES
        src/strndup.c
        src/main.c
        src/json_encode.c
        src/file_utils.c
        src/string_utils.c
        src/utf8proc/utf8proc.c
)

add_executable(libpostal ${LIBPOSTAL_SOURCES})
target_include_directories(libpostal PUBLIC src)
target_link_libraries(libpostal libpostal_la)

add_executable(bench src/bench.c)
target_include_directories(bench PUBLIC src)
target_link_libraries(bench libpostal_la)

set(ADDRESS_PARSER_SOURCES
    src/strndup.c
    src/address_parser_cli.c
    src/json_encode.c
    src/linenoise/linenoise.c
    src/string_utils.c
    src/utf8proc/utf8proc.c
)

add_executable(address_parser ${ADDRESS_PARSER_SOURCES})
target_include_directories(address_parser PUBLIC src)
target_link_libraries(address_parser libpostal_la)

set(NEAR_DUPE_TEST_SOURCES
    src/strndup.c
    src/near_dupe_test.c
    src/string_utils.c
    src/utf8proc/utf8proc.c
)
add_executable(near_dupe_test ${NEAR_DUPE_TEST_SOURCES})
target_include_directories(near_dupe_test PUBLIC src)
target_link_libraries(near_dupe_test libpostal_la)

set(BUILD_ADDRESS_DICTIONARY_SOURCES
        src/strndup.c
        src/address_dictionary_builder.c
        src/address_dictionary.c
        src/file_utils.c
        src/string_utils.c
        src/trie.c
        src/trie_search.c
        src/utf8proc/utf8proc.c
)
add_executable(build_address_dictionary ${BUILD_ADDRESS_DICTIONARY_SOURCES})
target_include_directories(build_address_dictionary PUBLIC src)
target_compile_definitions(build_address_dictionary PUBLIC HAVE_DIRENT_H=$<BOOL:${HAVE_DIRENT_H}>)
target_compile_definitions(build_address_dictionary PUBLIC LIBPOSTAL_DATA_DIR="${LIBPOSTAL_DATA_DIR}")


set(BUILD_NUMEX_TABLE_SOURCES
        src/strndup.c
        src/numex_table_builder.c
        src/numex.c
        src/file_utils.c
        src/string_utils.c
        src/tokens.c
        src/trie.c
        src/trie_search.c
        src/utf8proc/utf8proc.c
)
add_executable(build_numex_table ${BUILD_NUMEX_TABLE_SOURCES})
target_include_directories(build_numex_table PUBLIC src)
target_compile_definitions(build_numex_table PUBLIC HAVE_DIRENT_H=$<BOOL:${HAVE_DIRENT_H}>)
target_compile_definitions(build_numex_table PUBLIC LIBPOSTAL_DATA_DIR="${LIBPOSTAL_DATA_DIR}")


set(BUILD_TRANS_TABLE_SOURCES
        src/strndup.c
        src/transliteration_table_builder.c
        src/transliterate.c
        src/trie.c
        src/trie_search.c
        src/file_utils.c
        src/string_utils.c
        src/utf8proc/utf8proc.c
)
add_executable(build_trans_table ${BUILD_TRANS_TABLE_SOURCES})
target_include_directories(build_trans_table PUBLIC src)
target_compile_definitions(build_trans_table PUBLIC HAVE_DIRENT_H=$<BOOL:${HAVE_DIRENT_H}>)
target_compile_definitions(build_trans_table PUBLIC LIBPOSTAL_DATA_DIR="${LIBPOSTAL_DATA_DIR}")


set(ADDRESS_PARSER_TRAIN_SOURCES
        src/strndup.c
        src/address_parser_train.c
        src/address_parser.c
        src/address_parser_io.c
        src/averaged_perceptron.c
        src/crf.c
        src/crf_context.c
        src/sparse_matrix.c
        src/graph.c
        src/graph_builder.c
        src/float_utils.c
        src/averaged_perceptron_trainer.c
        src/crf_trainer.c
        src/crf_trainer_averaged_perceptron.c
        src/averaged_perceptron_tagger.c
        src/address_dictionary.c
        src/normalize.c
        src/numex.c
        src/features.c
        src/unicode_scripts.c
        src/transliterate.c
        src/trie.c
        src/trie_search.c
        src/trie_utils.c
        src/string_utils.c
        src/tokens.c
        src/file_utils.c
        src/shuffle.c
        src/utf8proc/utf8proc.c
        src/ngrams.c
)

add_executable(address_parser_train ${ADDRESS_PARSER_TRAIN_SOURCES})
target_include_directories(address_parser_train PUBLIC src)
target_link_libraries(address_parser_train libpostal_la)
target_compile_definitions(address_parser_train PUBLIC HAVE_SHUF=$<BOOL:${HAVE_SHUF}>)

set(LANGUAGE_CLASSIFIER_TRAIN_SOURCES
        src/strndup.c
        src/language_classifier_train.c
        src/language_classifier.c
        src/language_features.c
        src/language_classifier_io.c
        src/logistic_regression_trainer.c
        src/logistic_regression.c
        src/logistic.c
        src/sparse_matrix.c
        src/sparse_matrix_utils.c
        src/features.c
        src/minibatch.c
        src/float_utils.c
        src/stochastic_gradient_descent.c
        src/ftrl.c
        src/regularization.c
        src/cartesian_product.c
        src/normalize.c
        src/numex.c
        src/transliterate.c
        src/trie.c
        src/trie_search.c
        src/trie_utils.c
        src/address_dictionary.c
        src/string_utils.c
        src/file_utils.c
        src/utf8proc/utf8proc.c
        src/unicode_scripts.c
        src/shuffle.c
)

add_executable(language_classifier_train ${LANGUAGE_CLASSIFIER_TRAIN_SOURCES})
target_include_directories(language_classifier_train PUBLIC src)
target_link_libraries(language_classifier_train libscanner)
target_compile_definitions(language_classifier_train PUBLIC HAVE_SHUF=$<BOOL:${HAVE_SHUF}>)
target_compile_definitions(language_classifier_train PUBLIC HAVE_DIRENT_H=$<BOOL:${HAVE_DIRENT_H}>)
target_compile_definitions(language_classifier_train PUBLIC LIBPOSTAL_DATA_DIR="${LIBPOSTAL_DATA_DIR}")

set(LANGUAGE_CLASSIFIER_SOURCES
        src/strndup.c
        src/language_classifier_test.c
        src/language_classifier.c
        src/language_classifier_io.c
        src/language_features.c
        src/logistic_regression.c
        src/logistic.c
        src/sparse_matrix.c
        src/features.c
        src/minibatch.c
        src/float_utils.c
        src/normalize.c
        src/numex.c
        src/transliterate.c
        src/trie.c
        src/trie_search.c
        src/trie_utils.c
        src/address_dictionary.c
        src/string_utils.c
        src/file_utils.c
        src/utf8proc/utf8proc.c
        src/unicode_scripts.c
)
add_executable(language_classifier_test ${LANGUAGE_CLASSIFIER_SOURCES})
target_include_directories(language_classifier_test PUBLIC src)
target_link_libraries(language_classifier_test libscanner)
target_compile_definitions(language_classifier_test PUBLIC LIBPOSTAL_DATA_DIR="${LIBPOSTAL_DATA_DIR}")
target_compile_definitions(language_classifier_test PUBLIC HAVE_DIRENT_H=$<BOOL:${HAVE_DIRENT_H}>)
