cairo_trace_sources = [
  'trace.c',
]

shared_lib_ext = libcairo.full_path().split('.')[-1]

libcairotrace = library('cairo-trace', cairo_trace_sources,
  include_directories: [incbase, incsrc],
  dependencies: deps,
  link_args: extra_link_args,
  c_args: ['-DSHARED_LIB_EXT="@0@"'.format(shared_lib_ext), '-DHAVE_CONFIG_H'] + pthread_c_args,
)
