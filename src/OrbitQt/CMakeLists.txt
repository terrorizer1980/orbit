# Copyright (c) 2020 The Orbit Authors. All rights reserved.
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.

project(OrbitQt CXX)

add_library(OrbitQt OBJECT)

target_sources(
  OrbitQt
  PRIVATE AccessibilityAdapter.h
          AnnotatingSourceCodeDialog.h
          CallTreeViewItemModel.h
          CallTreeWidget.h
          CaptureOptionsDialog.h
          CustomSignalsTreeView.h
          CutoutWidget.h
          ElidedLabel.h
          FilterPanelWidget.h
          FilterPanelWidgetAction.h
          HistogramWidget.h
          MultipleOfSpinBox.h
          MultipleOfValidator.h
          orbitaboutdialog.h
          orbitdataviewpanel.h
          orbiteventiterator.h
          orbitglwidget.h
          orbitlivefunctions.h
          orbitmainwindow.h
          orbitsamplingreport.h
          orbittablemodel.h
          orbittreeview.h
          opengldetect.h
          resource.h
          TrackConfigurationWidget.h
          TrackTypeItemModel.h
          TutorialContent.h
          TutorialOverlay.h)

target_sources(
  OrbitQt
  PRIVATE AccessibilityAdapter.cpp
          AnnotatingSourceCodeDialog.cpp
          CallTreeViewItemModel.cpp
          CallTreeWidget.cpp
          CallTreeWidget.ui
          CaptureOptionsDialog.cpp
          CaptureOptionsDialog.ui
          ElidedLabel.cpp
          FilterPanelWidget.cpp
          FilterPanelWidgetAction.cpp
          HistogramWidget.cpp
          MultipleOfSpinBox.cpp
          MultipleOfValidator.cpp
          orbitaboutdialog.cpp
          orbitaboutdialog.ui
          orbitdataviewpanel.cpp
          orbitdataviewpanel.ui
          orbiteventiterator.cpp
          orbiteventiterator.ui
          orbitglwidget.cpp
          orbitlivefunctions.cpp
          orbitlivefunctions.ui
          orbitmainwindow.cpp
          orbitmainwindow.ui
          orbitsamplingreport.cpp
          orbitsamplingreport.ui
          orbittablemodel.cpp
          orbittreeview.cpp
          opengldetect.cpp
          TrackConfigurationWidget.cpp
          TrackConfigurationWidget.ui
          TrackTypeItemModel.cpp
          TutorialContent.cpp
          TutorialOverlay.cpp
          TutorialOverlay.ui
          ../../icons/orbiticons.qrc
          ../../images/orbitimages.qrc)

target_include_directories(OrbitQt PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_include_directories(OrbitQt PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)

target_link_libraries(
  OrbitQt
  PUBLIC  CaptureFileInfo
          ClientFlags
          ClientSymbols
          CodeViewer
          ConfigWidgets
          DisplayFormats
          MetricsUploader
          OrbitAccessibility
          OrbitGl
          OrbitGgp
          OrbitSshQt
          OrbitVersion
          QtUtils
          SessionSetup
          SourcePathsMapping
          SourcePathsMappingUI
          SymbolProvider
          UtilWidgets
          Qt5::Widgets
          Qt5::Core
          SyntaxHighlighter)

set_target_properties(OrbitQt PROPERTIES AUTOMOC ON)
set_target_properties(OrbitQt PROPERTIES AUTOUIC ON)
set_target_properties(OrbitQt PROPERTIES AUTORCC ON)
iwyu_add_dependency(OrbitQt)

add_executable(OrbitQtTests)

target_sources(OrbitQtTests
        PRIVATE AccessibilityAdapter.h
                CutoutWidget.h
                TutorialOverlay.h)
target_sources(OrbitQtTests
        PRIVATE AccessibilityAdapterTest.cpp
                AnnotatingSourceCodeDialogTest.cpp
                CallTreeViewItemModelTest.cpp
                HistogramWidgetTest.cpp
                MultipleOfValidatorTest.cpp
                TutorialOverlayTest.cpp
                TrackTypeItemModelTest.cpp)

target_include_directories(OrbitQtTests PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

target_link_libraries(OrbitQtTests
  PRIVATE OrbitGl
          OrbitQt
          Qt5::Test
          GTest::QtGuiMain)

if(WIN32)
  get_target_property(_qmake_executable Qt5::qmake IMPORTED_LOCATION)
  get_filename_component(_qt_bin_dir "${_qmake_executable}" DIRECTORY)
  find_program(WINDEPLOYQT_EXECUTABLE windeployqt HINTS "${_qt_bin_dir}")

  add_custom_command(
    TARGET OrbitQtTests
    POST_BUILD
    COMMAND "${CMAKE_COMMAND}" -E env PATH="${_qt_bin_dir}"
            "${WINDEPLOYQT_EXECUTABLE}" --pdb --no-angle "$<TARGET_FILE:OrbitQtTests>"
    COMMENT "Running windeployqt...")
endif()


set(test_properties "")

# QT_QPA_PLATFORM=offscreen is currently not supported under Windows.
# So if requested we disable the test all the way.
# This is mainly a work-around for the CI, especially the presubmit checks.
if (WIN32 AND "$ENV{QT_QPA_PLATFORM}" STREQUAL "offscreen")
  message(STATUS "Disabling OrbitQt-tests since they don't work in a headless setup")
  list(APPEND test_properties DISABLED TRUE)
endif()

if (NOT WIN32)
  # On Linux we can always run the tests with offscreen rendering. That won't do any harm
  # and it avoid test failures in headless environments like SSH-sessions.
  list(APPEND test_properties ENVIRONMENT QT_QPA_PLATFORM=offscreen)
endif()

register_test(OrbitQtTests PROPERTIES ${test_properties})
