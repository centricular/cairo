cairo_fdr_sources = [
  'fdr.c',
]

libcairofdr = library('cairo-fdr', cairo_fdr_sources,
  include_directories: [incbase, incsrc],
  c_args: ['-DHAVE_CONFIG_H'],
  dependencies: deps,
)
