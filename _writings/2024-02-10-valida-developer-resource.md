---
layout: post
author: Lita Foundation
title: "Developer Learning Materials on Valida zkVM"
excerpt: "A collection of essential materials, tips, and insights to streamline your learning process on Valida, accompanied by direct links to relevant GitHub issues"
image: "/assets/posts/2024-02-10-valida-developer-resource/preview.png"
---c

Delendum Research & Lita Foundation

**Objective**

We've curated a collection of essential materials, tips, and insights to streamline your learning process, accompanied by direct links to relevant GitHub issues. Join us on this journey as we delve into the intricacies of Valida zkVM development, providing the guidance you need to navigate the challenges and contribute effectively to the open-source community. 

Additionally, we are excited to announce that we are currently gathering interest for our open-source program, inviting developers and enthusiasts alike to contribute to the growth of the valida zkVM stack. Whether you're passionate about enhancing the core functionality of zkVM or envision creating innovative tools and applications on top of it, we welcome your collaboration. If you're interested, don't hesitate to reach out and connect with us. 

**Github organizations**

* [https://github.com/Plonky3](https://github.com/Plonky3)
* [https://github.com/lita-xyz](https://github.com/lita-xyz)
* [https://github.com/valida-xyz/](https://github.com/valida-xyz/) 

**Valida resources**

* ISA reference: [https://github.com/valida-xyz/valida-compiler/issues/2](https://github.com/valida-xyz/valida-compiler/issues/2) 

**Valida LLVM resources**

* The virtual register lowering pass: [https://github.com/lita-xyz/llvm-valida/commit/60f352fab03b138cd3f97882a6e8c0392acd3daa](https://github.com/lita-xyz/llvm-valida/commit/60f352fab03b138cd3f97882a6e8c0392acd3daa) 
* The calling convention: 
    * [https://github.com/lita-xyz/llvm-valida/blob/97f7f83624b4805aef1b7019e6fb3311499081ac/llvm/lib/Target/Delendum/DelendumCallingConv.td](https://github.com/lita-xyz/llvm-valida/blob/97f7f83624b4805aef1b7019e6fb3311499081ac/llvm/lib/Target/Delendum/DelendumCallingConv.td) 
* Registers:
    * [https://github.com/lita-xyz/llvm-valida/blob/97f7f83624b4805aef1b7019e6fb3311499081ac/llvm/lib/Target/Delendum/DelendumRegisterInfo.td](https://github.com/lita-xyz/llvm-valida/blob/97f7f83624b4805aef1b7019e6fb3311499081ac/llvm/lib/Target/Delendum/DelendumRegisterInfo.td) 
* Design docs: [https://github.com/lita-xyz/llvm-valida/blob/hliao/main/delendum-dev/DESIGN.md](https://github.com/lita-xyz/llvm-valida/blob/hliao/main/delendum-dev/DESIGN.md) 

**LLVM resources**

* Build instructions (CMake): [https://llvm.org/docs/CMake.html](https://llvm.org/docs/CMake.html)
* General introductions:
    * [https://llvm.org/pubs/2008-10-04-ACAT-LLVM-Intro.html](https://llvm.org/pubs/2008-10-04-ACAT-LLVM-Intro.html)
    * [https://aosabook.org/en/v1/llvm.html](https://aosabook.org/en/v1/llvm.html)
    * [https://llvm.org/pubs/2004-01-30-CGO-LLVM.html](https://llvm.org/pubs/2004-01-30-CGO-LLVM.html)
* Doxygen: [https://llvm.org/doxygen](https://llvm.org/doxygen) 
* TableGen
    * Overview:  [https://llvm.org/docs/TableGen/index.html](https://llvm.org/docs/TableGen/index.html) 
    * Reference: [https://llvm.org/docs/TableGen/ProgRef.html](https://llvm.org/docs/TableGen/ProgRef.html)
    * Backends: [https://llvm.org/docs/TableGen/BackEnds.html](https://llvm.org/docs/TableGen/BackEnds.html) 
* LLVM IR reference: [https://llvm.org/docs/LangRef.html](https://llvm.org/docs/LangRef.html) 
* Target-independent codegen: [https://llvm.org/docs/CodeGenerator.html](https://llvm.org/docs/CodeGenerator.html) 
* Writing an LLVM (codegen) pass: [https://llvm.org/docs/WritingAnLLVMPass.html](https://llvm.org/docs/WritingAnLLVMPass.html) 
* LLVM analysis & transform passes: [https://llvm.org/docs/Passes.html](https://llvm.org/docs/Passes.html) 
* Writing an LLVM backend: [https://llvm.org/docs/WritingAnLLVMBackend.html](https://llvm.org/docs/WritingAnLLVMBackend.html) 
* Creating an LLVM backend tutorial: [http://jonathan2251.github.io/lbd/](http://jonathan2251.github.io/lbd/) 
* LLVM Linker (LLD): [https://lld.llvm.org/](https://lld.llvm.org/) 
* DWARF intro: [https://dwarfstd.org/doc/Debugging%20using%20DWARF-2012.pdf](https://dwarfstd.org/doc/Debugging%20using%20DWARF-2012.pdf) 
* ELF intro: [https://en.wikipedia.org/wiki/Executable_and_Linkable_Format](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) 
* DWARF CIE exception frames: [https://refspecs.linuxfoundation.org/LSB_3.0.0/LSB-Core-generic/LSB-Core-generic/ehframechpt.html](https://refspecs.linuxfoundation.org/LSB_3.0.0/LSB-Core-generic/LSB-Core-generic/ehframechpt.html) 
* Extending LLVM: [https://llvm.org/docs/ExtendingLLVM.html](https://llvm.org/docs/ExtendingLLVM.html) 
* LLVM testing: [https://llvm.org/docs/TestingGuide.html](https://llvm.org/docs/TestingGuide.html) 
* LLVM test suite guide: [https://llvm.org/docs/TestSuiteGuide.html](https://llvm.org/docs/TestSuiteGuide.html) 
* Building libc (host): [https://libc.llvm.org/full_host_build.html](https://libc.llvm.org/full_host_build.html) 
* Building libc (cross): [https://libc.llvm.org/full_cross_build.html](https://libc.llvm.org/full_cross_build.html) 
* Building libcxx: [https://libcxx.llvm.org/BuildingLibcxx.html](https://libcxx.llvm.org/BuildingLibcxx.html) 
* Example of defining a calling convention: [https://github.com/Orbis-Tertius/llvm-project/commit/6e2a222b773f1d2e183f708a97fa29de607544d0](https://github.com/Orbis-Tertius/llvm-project/commit/6e2a222b773f1d2e183f708a97fa29de607544d0) 
    * [https://github.com/Orbis-Tertius/llvm-project/blob/6e2a222b773f1d2e183f708a97fa29de607544d0/llvm/lib/Target/TinyRAM/TinyRAMCallingConv.td](https://github.com/Orbis-Tertius/llvm-project/blob/6e2a222b773f1d2e183f708a97fa29de607544d0/llvm/lib/Target/TinyRAM/TinyRAMCallingConv.td) 
    * [https://github.com/Orbis-Tertius/llvm-project/commit/6e2a222b773f1d2e183f708a97fa29de607544d0#diff-a24b466670a7885b6315c0aad1370b6306b731f7bbb7c5b8fabe29f27a150acc](https://github.com/Orbis-Tertius/llvm-project/commit/6e2a222b773f1d2e183f708a97fa29de607544d0#diff-a24b466670a7885b6315c0aad1370b6306b731f7bbb7c5b8fabe29f27a150acc) 
* Example of AsmInfo class definition: [https://github.com/Orbis-Tertius/llvm-project/commit/5004c4bd0b58fa411b2001a785eb544f2175e80f](https://github.com/Orbis-Tertius/llvm-project/commit/5004c4bd0b58fa411b2001a785eb544f2175e80f) 
* Example of ELF relocation support: [https://github.com/Orbis-Tertius/llvm-project/commit/c0eb8bc1c15caf2f0c00cc0aa2621f503918e435](https://github.com/Orbis-Tertius/llvm-project/commit/c0eb8bc1c15caf2f0c00cc0aa2621f503918e435) 
* Example of I/O primitives: [https://github.com/Orbis-Tertius/llvm-project/commit/b28ca360b0f518a784427cf3bc18c14ed995f3b0](https://github.com/Orbis-Tertius/llvm-project/commit/b28ca360b0f518a784427cf3bc18c14ed995f3b0) 
* Example of zeroing struct alignment: [https://github.com/Orbis-Tertius/llvm-project/commit/6eb86bf281d733b0cfb52e161c509bda9d8c015d](https://github.com/Orbis-Tertius/llvm-project/commit/6eb86bf281d733b0cfb52e161c509bda9d8c015d) 

**Runtime system resources**



* C runtime initialization: [https://www.embecosm.com/appnotes/ean9/html/ch05s02.html](https://www.embecosm.com/appnotes/ean9/html/ch05s02.html)
* Rust runtime initialization: [https://github.com/rust-lang/rust/blob/33916307780495fe311fe9c080b330d266f35bfb/src/libstd/rt.rs#L43](https://github.com/rust-lang/rust/blob/33916307780495fe311fe9c080b330d266f35bfb/src/libstd/rt.rs#L43) 

**EVM resources**



* EVM intro: [https://ethereum.org/en/developers/docs/evm/](https://ethereum.org/en/developers/docs/evm/) 
* Yellowpaper: [https://ethereum.github.io/yellowpaper/paper.pdf](https://ethereum.github.io/yellowpaper/paper.pdf) 
* Opcode reference: [https://www.ethervm.io/](https://www.ethervm.io/) 

**ELF resources**



* Visual: [https://en.wikipedia.org/wiki/Executable_and_Linkable_Format#/media/File:ELF_Executable_and_Linkable_Format_diagram_by_Ange_Albertini.png](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format#/media/File:ELF_Executable_and_Linkable_Format_diagram_by_Ange_Albertini.png) 
* System V ABI
    * Intro: [https://wiki.osdev.org/System_V_ABI](https://wiki.osdev.org/System_V_ABI) 
    * Spec: [https://www.sco.com/developers/devspecs/gabi41.pdf](https://www.sco.com/developers/devspecs/gabi41.pdf) 
* RISC-V ELF spec: [https://github.com/riscv-non-isa/riscv-elf-psabi-doc/blob/master/riscv-elf.adoc](https://github.com/riscv-non-isa/riscv-elf-psabi-doc/blob/master/riscv-elf.adoc) 

__________________________________

_If you’re interested in further discussions on this topic or working together on this subject, please reach out to us at research@delendum.xyz._
