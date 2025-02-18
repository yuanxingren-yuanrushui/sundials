# ---------------------------------------------------------------
# Programmer(s): Radu Serban @ LLNL
# ---------------------------------------------------------------
# SUNDIALS Copyright Start
# Copyright (c) 2002-2022, Lawrence Livermore National Security
# and Southern Methodist University.
# All rights reserved.
#
# See the top-level LICENSE and NOTICE files for details.
#
# SPDX-License-Identifier: BSD-3-Clause
# SUNDIALS Copyright End
# ---------------------------------------------------------------
# CMakeLists.txt file for FIDA PThread examples
# ---------------------------------------------------------------

# Example lists are tuples "name\;args\;type" where the type is
# 'develop' for examples excluded from 'make test' in releases

# Only include tests if Pthreads is enabled
if(PTHREADS_FOUND)

  # Examples using SUNDIALS linear solvers
  set(FIDA_examples
    "fidaRoberts_dns_pthreads\;\;develop"
    )

endif()

# Specify libraries to link against
set(FNVECS_LIB sundials_fnvecpthreads)

# Only static FCMIX libraries are available
set(FIDA_LIB sundials_fida${_STATIC_LIB_SUFFIX})

# Set-up linker flags and link libraries
set(SUNDIALS_LIBS ${FIDA_LIB} ${FNVECS_LIB} ${EXE_EXTRA_LINK_LIBS})


# Add the build and install targets for each example
foreach(example_tuple ${FIDA_examples})

  # parse the example tuple
  list(GET example_tuple 0 example)
  list(GET example_tuple 1 example_args)
  list(GET example_tuple 2 example_type)

  # example source files
  add_executable(${example} ${example}.f)

  set_target_properties(${example} PROPERTIES FOLDER "Examples")

  # add example to regression tests
  sundials_add_test(${example} ${example}
    TEST_ARGS ${example_args}
    ANSWER_DIR ${CMAKE_CURRENT_SOURCE_DIR}
    ANSWER_FILE ${example}.out
    EXAMPLE_TYPE ${example_type})

  # libraries to link against
  target_link_libraries(${example} ${SUNDIALS_LIBS} ${CMAKE_THREAD_LIBS_INIT})

   # install example source and out files
  if(EXAMPLES_INSTALL)
    install(FILES ${example}.f ${example}.out
      DESTINATION ${EXAMPLES_INSTALL_PATH}/ida/fcmix_pthreads)
  endif()

endforeach(example_tuple ${FIDA_examples})

# create Makfile and CMakeLists.txt for examples
if(EXAMPLES_INSTALL)

  # Install the README file
  install(FILES README DESTINATION ${EXAMPLES_INSTALL_PATH}/ida/fcmix_pthreads)

  # Prepare substitution variables for Makefile and/or CMakeLists templates
  set(SOLVER "IDA")
  set(SOLVER_LIB "sundials_ida")
  set(SOLVER_FLIB "sundials_fida")

  examples2string(FIDA_examples EXAMPLES)

  # Regardless of the platform we're on, we will generate and install
  # CMakeLists.txt file for building the examples. This file  can then
  # be used as a template for the user's own programs.

  # generate CMakelists.txt in the binary directory
  configure_file(
    ${PROJECT_SOURCE_DIR}/examples/templates/cmakelists_pthreads_F77_ex.in
    ${PROJECT_BINARY_DIR}/examples/ida/fcmix_pthreads/CMakeLists.txt
    @ONLY
    )

  # install CMakelists.txt
  install(
    FILES ${PROJECT_BINARY_DIR}/examples/ida/fcmix_pthreads/CMakeLists.txt
    DESTINATION ${EXAMPLES_INSTALL_PATH}/ida/fcmix_pthreads
    )

  # On UNIX-type platforms, we also  generate and install a makefile for
  # building the examples. This makefile can then be used as a template
  # for the user's own programs.

  if(UNIX)
    # generate Makefile and place it in the binary dir
    configure_file(
      ${PROJECT_SOURCE_DIR}/examples/templates/makefile_pthreads_F77_ex.in
      ${PROJECT_BINARY_DIR}/examples/ida/fcmix_pthreads/Makefile_ex
      @ONLY
      )
    # install the configured Makefile_ex as Makefile
    install(
      FILES ${PROJECT_BINARY_DIR}/examples/ida/fcmix_pthreads/Makefile_ex
      DESTINATION ${EXAMPLES_INSTALL_PATH}/ida/fcmix_pthreads
      RENAME Makefile
      )
  endif()

  # add test_install target
  sundials_add_test_install(ida fcmix_pthreads)

endif()
