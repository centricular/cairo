cairo_boilerplate_sources = [
  'cairo-boilerplate-getopt.c',
  'cairo-boilerplate-system.c',
  'cairo-boilerplate.c',
]

if feature_conf.get('CAIRO_HAS_XLIB_SURFACE', 0) == 1
  cairo_boilerplate_sources += ['cairo-boilerplate-xlib.c']
endif

incboilerplate = include_directories('.')

cairo_boilerplate_constructors = custom_target('cairo-boilerplate-constructors.c',
  input: files(cairo_boilerplate_sources),
  output: 'cairo-boilerplate-constructors.c',
  command: [python3, files('make-cairo-boilerplate-constructors.py')[0], '@OUTPUT@', '@INPUT@'])

libcairoboilerplate = static_library('cairoboilerplate', cairo_boilerplate_sources + [cairo_boilerplate_constructors],
  c_args: ['-DHAVE_CONFIG_H'],
  include_directories: [incbase, incsrc],
  dependencies: deps,
  link_with: [libcairo],
)
