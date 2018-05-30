cairo_boilerplate_sources = [
  'cairo-boilerplate-getopt.c',
  'cairo-boilerplate-system.c',
  'cairo-boilerplate.c',
]

if feature_conf.get('CAIRO_HAS_XLIB_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-xlib.c']
endif

if feature_conf.get('CAIRO_HAS_QT_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-qt.cpp']
endif

if feature_conf.get('CAIRO_HAS_QUARTZ_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-quartz.c']
endif

if feature_conf.get('CAIRO_HAS_XCB_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-xcb.c']
endif

if feature_conf.get('CAIRO_HAS_WIN32_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-win32.c', 'cairo-boilerplate-win32-printing.c']
endif

if feature_conf.get('CAIRO_HAS_DRM_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-drm.c']
endif

if feature_conf.get('CAIRO_HAS_COGL_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-cogl.c']
endif

if feature_conf.get('CAIRO_HAS_DIRECTFB_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-directfb.c']
endif

if feature_conf.get('CAIRO_HAS_PDF_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-pdf.c']
endif

if feature_conf.get('CAIRO_HAS_PS_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-ps.c']
endif

if feature_conf.get('CAIRO_HAS_SVG_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-svg.c']
endif

if feature_conf.get('CAIRO_HAS_VG_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-vg.c']
endif

if feature_conf.get('CAIRO_HAS_SCRIPT_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-script.c']
endif

incboilerplate = include_directories('.')

cairo_boilerplate_constructors = custom_target('cairo-boilerplate-constructors.c',
  input: files(cairo_boilerplate_sources),
  output: 'cairo-boilerplate-constructors.c',
  command: [python3, files('make-cairo-boilerplate-constructors.py')[0], '@OUTPUT@', '@INPUT@'])

libcairoboilerplate = static_library('cairoboilerplate', cairo_boilerplate_sources + [cairo_boilerplate_constructors],
  c_args: ['-DHAVE_CONFIG_H'],
  cpp_args: ['-DHAVE_CONFIG_H'],
  include_directories: [incbase, incsrc],
  dependencies: deps,
  link_with: [libcairo],
)
