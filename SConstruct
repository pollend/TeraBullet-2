#!python
import os, subprocess,sys

# Local dependency paths, adapt them to your setup
godot_headers_path = "godot-cpp/godot_headers/"

# Try to detect the host platform automatically
# This is used if no `platform` argument is passed
if sys.platform.startswith('linux'):
    host_platform = 'linux'
elif sys.platform == 'darwin':
    host_platform = 'osx'
elif sys.platform == 'win32':
    host_platform = 'windows'
else:
    raise ValueError('Could not detect platform automatically, please specify with platform=<platform>')

opts = Variables([], ARGUMENTS)
opts.Add(EnumVariable('platform', 'Target platform', host_platform, ('linux', 'osx', 'windows')))
opts.Add(EnumVariable('bits', 'Target platform bits', 'default', ('default', '32', '64')))
opts.Add(BoolVariable('use_llvm', 'Use the LLVM compiler - only effective when targeting Linux', False))
opts.Add(BoolVariable('use_mingw', 'Use the MinGW compiler - only effective on Windows', False))
opts.Add(EnumVariable('target', 'Compilation target', 'debug', ('debug', 'release')))
opts.Add(BoolVariable('test', 'Run tests', False))


# This makes sure to keep the session environment variables on windows,
# that way you can run scons in a vs 2017 prompt and it will find all the required tools
env = Environment()
Export('env')

opts.Update(env)
Help(opts.GenerateHelpText(env))

if ARGUMENTS.get("use_llvm", "no") == "yes":
    env["CXX"] = "clang++"

def add_sources(sources, directory):
    for file in os.listdir(directory):
        if file.endswith('.cpp'):
            sources.append(directory + '/' + file)

is64 = sys.maxsize > 2**32
if env['bits'] == 'default':
    env['bits'] = '64' if is64 else '32'

if env['platform'] == 'linux':
    if env['use_llvm']:
        env['CXX'] = 'clang++'

    env.Append(CCFLAGS=['-fPIC', '-g', '-std=c++14', '-Wwrite-strings'])
    env.Append(LINKFLAGS=["-Wl,-R,'$$ORIGIN'"])

    if env['target'] == 'debug':
        env.Append(CCFLAGS=['-Og'])
    elif env['target'] == 'release':
        env.Append(CCFLAGS=['-O3'])

    if env['bits'] == '64':
        env.Append(CCFLAGS=['-m64'])
        env.Append(LINKFLAGS=['-m64'])
    elif env['bits'] == '32':
        env.Append(CCFLAGS=['-m32'])
        env.Append(LINKFLAGS=['-m32'])

elif env['platform'] == 'osx':
    if env['bits'] == '32':
        raise ValueError('Only 64-bit builds are supported for the macOS target.')

    env.Append(CCFLAGS=['-g', '-std=c++14', '-arch', 'x86_64'])
    env.Append(LINKFLAGS=['-arch', 'x86_64', '-framework', 'Cocoa', '-Wl,-undefined,dynamic_lookup'])

    if env['target'] == 'debug':
        env.Append(CCFLAGS=['-Og'])
    elif env['target'] == 'release':
        env.Append(CCFLAGS=['-O3'])

elif env['platform'] == 'windows':
    # This makes sure to keep the session environment variables on Windows
    # This way, you can run SCons in a Visual Studio 2017 prompt and it will find all the required tools
    if env['bits'] == '64':
        env.Override(os.environ)
        env.Append(TARGET_ARCH='amd64')
    elif env['bits'] == '32':
        env.Override(os.environ)
        env.Append(TARGET_ARCH='x86')

    if host_platform == 'windows' and not env['use_mingw']:
        # MSVC
        env.Append(LINKFLAGS=['/WX'])
        if env['target'] == 'debug':
            env.Append(CCFLAGS=['/EHsc', '/D_DEBUG', '/MDd'])
        elif env['target'] == 'release':
            env.Append(CCFLAGS=['/O2', '/EHsc', '/DNDEBUG', '/MD'])
    else:
        # MinGW
        if env['bits'] == '64':
            env['CXX'] = 'x86_64-w64-mingw32-g++'
        elif env['bits'] == '32':
            env['CXX'] = 'i686-w64-mingw32-g++'

        env.Append(CCFLAGS=['-g', '-O3', '-std=c++14', '-Wwrite-strings'])
        env.Append(LINKFLAGS=['--static', '-Wl,--no-undefined', '-static-libgcc', '-static-libstdc++'])


Export('env')

SConscript("swig-src/SCsub")


# env.Append(CPPPATH=['.', 'library/include', godot_headers_path, 'godot-cpp/include/','godot-cpp/include/core/'])
# env.Append(LIBPATH=[Dir('godot-cpp/bin'), Dir('bin/{}'.format(env['platform']))])
# env.Append(LIBS=["godot-cpp.{}.{}.{}".format(env['platform'],env['target'],env['bits'])])

# sources = []
# add_sources(sources, "library/src")

# library = env.SharedLibrary(target=env['build_root'].abspath + '/{}/QSys'.format(env['platform']), source=sources)
# # SConscript('test/SConstruct',exports='env')
# Default(library)