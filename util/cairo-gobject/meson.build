cairo_gobject_sources = [
  'cairo-gobject-enums.c',
  'cairo-gobject-structs.c',
]

libcairogobject = library('cairo-gobject', cairo_gobject_sources,
  c_args: ['-DHAVE_CONFIG_H'],
  include_directories: [incbase, incsrc],
  dependencies: [glib_dep, gobject_dep],
  link_with: [libcairo],
)
