cairoscript_interpreter_sources = [
  'cairo-script-file.c',
  'cairo-script-hash.c',
  'cairo-script-interpreter.c',
  'cairo-script-objects.c',
  'cairo-script-operators.c',
  'cairo-script-scanner.c',
  'cairo-script-stack.c',
]

csi_replay_sources = [
  'csi-replay.c',
]

csi_exec_sources = [
  'csi-exec.c',
]

csi_trace_sources = [
  'csi-trace.c',
]

inccairoscript = include_directories('.')

libcairoscript = static_library('libcairo-script', cairoscript_interpreter_sources,
  c_args: ['-DHAVE_CONFIG_H'],
  include_directories: [incbase, incsrc],
  dependencies: deps,
  link_with: [libcairo],
)

csi_replay_exe = executable('csi-replay', csi_replay_sources,
  c_args: ['-DHAVE_CONFIG_H'],
  include_directories: [incbase, incsrc],
  link_with: [libcairoscript],
)

csi_exec_exe = executable('csi-exec', csi_exec_sources,
  c_args: ['-DHAVE_CONFIG_H'],
  include_directories: [incbase, incsrc],
  link_with: [libcairoscript],
)

if feature_conf.get('CAIRO_HAS_SCRIPT_SURFACE', 0) == 1
  csi_trace_exe = executable('csi-trace', csi_trace_sources,
    c_args: ['-DHAVE_CONFIG_H'],
    include_directories: [incbase, incsrc],
    link_with: [libcairoscript],
  )
endif
