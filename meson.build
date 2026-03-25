project(
    'leechcore-meson',
    [ 'c' ],
    version: '2.22',
    default_options: [
        'b_ndebug=true'
    ]
)

cc = meson.get_compiler('c')

leechcore_src = files(
    'leechcore/ob/ob_bytequeue.c',
    'leechcore/ob/ob_core.c',
    'leechcore/ob/ob_map.c',
    'leechcore/ob/ob_set.c',
    'leechcore/ob/ob.h',
    
    'leechcore/util.c',
    'leechcore/util.h',
    'leechcore/charutil.c',
    'leechcore/charutil.h',

    'leechcore/leechcore.c',
    'leechcore/leechcore.h',
    'leechcore/leechrpc.h',
    'leechcore/leechcore_internal.h',
    'leechcore/leechrpcclient.c',
    'leechcore/memmap.c',
    'leechcore/oscompatibility.c',
    'leechcore/oscompatibility.h',

    'leechcore/leechcore_device.h',
    'leechcore/device_file.c',
    'leechcore/device_fpga.c',
    'leechcore/device_hibr.c',
    'leechcore/device_pmem.c',
    'leechcore/device_tmd.c',
    'leechcore/device_usb3380.c',
    'leechcore/device_vmm.c',
    'leechcore/device_vmware.c',

    'leechcore/version.h',
)

leechcore_deps = []
leechcore_c_args = []

if target_machine.kernel() == 'nt'
    leechcore_src += files(
        'leechcore/leechrpcshared.c',
    )

    leechcore_deps += [
        cc.find_library('Setupapi'),
        cc.find_library('Winusb'),
        cc.find_library('Ws2_32'),
        cc.find_library('Rpcrt4'),
        cc.find_library('Secur32'),
        cc.find_library('Credui'),
    ]

    windows = import('windows')
    if windows.found()
        leechcore_src += windows.compile_resources('leechcore/leechcore.rc')
    endif

    midl = find_program('midl')

    midl_args = [
        '@EXTRA_ARGS@',
        '@INPUT@',
        '/h', '@BASENAME@_h.h',
        '/cstub', '@BASENAME@_c.c',
        '/sstub', '@BASENAME@_s.c',
        '/out', '@BUILD_DIR@',
        '/env', 'x64',
        '/W1',
        '/char', 'signed',
        '/nologo',
        '/Zp8',
    ]

    midl_gen_client = generator(midl,
        output: [ '@BASENAME@_c.c', '@BASENAME@_h.h' ],
        arguments: midl_args + [
            '/server', 'none',
            '/client', 'stub'
        ]
    )

    midl_gen_server = generator(midl,
        output: [ '@BASENAME@_s.c', '@BASENAME@_h.h' ],
        arguments: midl_args + [
            '/server', 'stub',
            '/client', 'none'
        ]
    )

    leechcore_src += midl_gen_client.process(
        'leechcore/leechrpc.idl',
    )
else
    leechcore_c_args += [
        '-DLINUX',
        '-D_GNU_SOURCE',
        '-Wall',
        '-Wno-multichar',
        '-Wno-unused-result',
        '-Wno-unused-variable',
        '-Wno-unused-value',
        '-Wno-pointer-to-int-cast',
        '-Wno-int-to-pointer-cast',
        '-Wno-unused-but-set-variable',
    ]

    leechcore_deps += [
        dependency('libusb-1.0'),
        dependency('threads'),
    ]
endif

leechcore_includes = include_directories('includes')

leechcore_lib = library(
    'leechcore',
    leechcore_src,
    include_directories: leechcore_includes,
    dependencies: leechcore_deps,
    c_args: leechcore_c_args,
    install: true,
)

leechcore_dep = declare_dependency(
    link_with: leechcore_lib,
    include_directories: leechcore_includes
)

if get_option('leechagent')
    leechagent_deps = [ leechcore_dep ]

    if target_machine.kernel() == 'nt'
        leechagent_src = files(
            'leechagent/leechagent_proc.h',
            'leechagent/leechagent_procchild.c',
            'leechagent/leechagent_procparent.c',
            'leechagent/leechagent_rpc.c',
            'leechagent/leechagent_rpc.h',
            'leechagent/leechagent_svc.c',
            'leechagent/leechagent_svc.h',
            'leechagent/leechagent.c',
            'leechagent/leechagent.h',
            'leechagent/leechrpc.h',
            'leechagent/leechrpcserver.c',
            'leechagent/leechrpcshared.c',
            'leechagent/util.c',
            'leechagent/util.h',

            'leechagent/version.h',
        )

        leechagent_deps += [
            cc.find_library('Rpcrt4'),
            cc.find_library('Secur32'),
        ]

        leechagent_src += midl_gen_server.process(
            'leechagent/leechrpc.idl',
        )
    else
        leechagent_src = files(
            'leechagent_linux/leechagent_proc.h',
            'leechagent_linux/leechagent_rpc.c',
            'leechagent_linux/leechagent_rpc.h',

            'leechagent_linux/leechagent.c',
            'leechagent_linux/leechagent.h',
            'leechagent_linux/leechrpc.h',
            'leechagent_linux/leechrpcserver.c',
            'leechagent_linux/leechrpcshared.c',
            'leechagent_linux/oscompatibility.c',
            'leechagent_linux/oscompatibility.h',
        )
    endif

    leechagent = executable(
        'leechagent',
        leechagent_src,
        dependencies: leechagent_deps,
        c_args: leechcore_c_args,
        install: true,
    )
endif
