# This file is part of the Black Magic Debug project.
#
# Copyright (C) 2023 1BitSquared <info@1bitsquared.com>
# Written by Rafael Silva <perigoso@riseup.net>
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
# 1. Redistributions of source code must retain the above copyright notice, this
#    list of conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above copyright notice,
#    this list of conditions and the following disclaimer in the documentation
#    and/or other materials provided with the distribution.
#
# 3. Neither the name of the copyright holder nor the names of its
#    contributors may be used to endorse or promote products derived from
#    this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
# CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
# OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

probe_f072_includes = include_directories('.')

probe_f072_sources = files(
	'platform.c',
	'atomic.c',
)

probe_f072_args = [
	'-DDFU_SERIAL_LENGTH=13',
]

probe_f072_link_args = [
	'-L@0@'.format(meson.current_source_dir()),
	'-T@0@'.format('stm32f07xzb.ld'),
]

probe_f072_dependencies = [
	platform_common,
	platform_stm32f0,
	platform_stm32_swo,
	platform_stm32_swo_manchester,
]

probe_host = declare_dependency(
	include_directories: probe_f072_includes,
	sources: probe_f072_sources,
	compile_args: probe_f072_args,
	link_args: probe_f072_link_args,
	dependencies: probe_f072_dependencies,
)

summary(
	{
		'Name': 'STM32F072',
		'Platform': 'STM32F0',
		'Bootloader': 'OEM ST Bootloader',
	},
	section: 'Probe',
)
