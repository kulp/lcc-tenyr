%{
#define REGS(_) \
        _(A) \
        _(B) \
        _(C) \
        _(D) \
        _(E) \
        _(F) \
        _(G) \
        _(H) \
        _(I) \
        _(J) \
        _(K) \
        _(L) \
        _(M) \
        _(N) \
        _(O) \
        _(P) \
//

enum TenyrRegister {
#define REG_enum(x) REG_##x,
        REGS(REG_enum)
        REG_max
};

const static char reg_names[][2] = {
#define REG_name(x) #x,
        REGS(REG_name)
        0
};

#include "c.h"
#define NODEPTR_TYPE Node
#define OP_LABEL(p) ((p)->op)
#define LEFT_CHILD(p) ((p)->kids[0])
#define RIGHT_CHILD(p) ((p)->kids[1])
#define STATE_LABEL(p) ((p)->x.state)
static void address(Symbol, Symbol, long);
static void blkfetch(int, int, int, int);
static void blkloop(int, int, int, int, int, int[]);
static void blkstore(int, int, int, int);
static void defaddress(Symbol);
static void defconst(int, int, Value);
static void defstring(int, char *);
static void defsymbol(Symbol);
static void doarg(Node);
static void emit2(Node);
static void export(Symbol);
static void clobber(Node);
static void function(Symbol, Symbol [], Symbol [], int);
static void global(Symbol);
static void import(Symbol);
static void local(Symbol);
static void progbeg(int, char **);
static void progend(void);
static void segment(int);
static void space(int);
static void target(Node);
static Symbol intreg[32];

static Symbol intregw;

%}
%start stmt
%term CNSTI1=1045
%term CNSTP1=1047
%term CNSTU1=1046
 
%term ARGI1=1061
%term ARGP1=1063
%term ARGU1=1062

%term ASGNB=57
%term ASGNI1=1077
%term ASGNP1=1079
%term ASGNU1=1078

%term INDIRB=73
%term INDIRI1=1093
%term INDIRP1=1095
%term INDIRU1=1094

%term CVII1=1157
%term CVIU1=1158

%term CVPP1=1175
%term CVPU1=1174

%term CVUI1=1205
%term CVUP1=1207
%term CVUU1=1206

%term NEGI1=1221

%term CALLI1=1237
%term CALLP1=1239
%term CALLU1=1238
%term CALLV=216

%term RETI1=1269
%term RETP1=1271
%term RETU1=1270
%term RETV=248

%term ADDRGP1=1287

%term ADDRFP1=1303

%term ADDRLP1=1319

%term ADDI1=1333
%term ADDP1=1335
%term ADDU1=1334

%term SUBI1=1349
%term SUBP1=1351
%term SUBU1=1350

%term LSHI1=1365
%term LSHU1=1366

%term MODI1=1381
%term MODU1=1382

%term RSHI1=1397
%term RSHU1=1398

%term BANDI1=1413
%term BANDU1=1414

%term BCOMI1=1429
%term BCOMU1=1430

%term BORI1=1445
%term BORU1=1446

%term BXORI1=1461
%term BXORU1=1462

%term DIVI1=1477
%term DIVU1=1478

%term MULI1=1493
%term MULU1=1494

%term EQI1=1509
%term EQU1=1510

%term GEI1=1525
%term GEU1=1526

%term GTI1=1541
%term GTU1=1542

%term LEI1=1557
%term LEU1=1558

%term LTI1=1573
%term LTU1=1574

%term NEI1=1589
%term NEU1=1590

%term JUMPV=584

%term LABELV=600

%term LOADB=233
%term LOADI1=1253
%term LOADP1=1255
%term LOADU1=1254

%term VREGP=711
%%

reg:  INDIRI1(VREGP)        "# read register\n"
reg:  INDIRP1(VREGP)        "# read register\n"
reg:  INDIRU1(VREGP)        "# read register\n"

stmt: ASGNI1(VREGP,reg)     "# write register\n"
stmt: ASGNP1(VREGP,reg)     "# write register\n"
stmt: ASGNU1(VREGP,reg)     "# write register\n"

con12: CNSTI1               "%a" range(a, -2048, 2047)
con12: CNSTU1               "%a" range(a,     0, 2047)
con12: CNSTP1               "%a" range(a, -2048, 2047)
con20: CNSTI1               "%a" range(a, -(1 << 19), (1 << 19) - 1)
con32: CNSTI1               "%c <- (%a  >> 12); %c <- %c ^^ (%a & 0xfff)\n" 2
con32: CNSTU1               "%c <- (%a >>> 12); %c <- %c ^^ (%a & 0xfff)\n" 2
reg: con32                  "%0"

reg: CNSTI1                 "# reg\n" range(a, 0, 0)
reg: CNSTU1                 "# reg\n" range(a, 0, 0)
reg: CNSTP1                 "# reg\n" range(a, 0, 0)

rhs: ADDI1(reg,rc12)        "%0  +  %1"
rhs: ADDI1(rc12,reg)        "%0  +  %1"
rhs: SUBI1(reg,rc12)        "%0  -  %1"
rhs: SUBI1(rc12,reg)        "%0  -  %1"
rhs: MULI1(reg,rc12)        "%0  *  %1"
rhs: MULI1(rc12,reg)        "%0  *  %1"
rhs: BORI1(reg,rc12)        "%0  |  %1"
rhs: BORI1(rc12,reg)        "%0  |  %1"
rhs: BANDI1(reg,rc12)       "%0  &  %1"
rhs: BANDI1(rc12,reg)       "%0  &  %1"
rhs: BXORI1(reg,rc12)       "%0  ^  %1"
rhs: BXORI1(rc12,reg)       "%0  ^  %1"
rhs: LSHI1(reg,rc12)        "%0  << %1"
rhs: LSHI1(rc12,reg)        "%0  << %1"
rhs: RSHI1(reg,rc12)        "%0  >> %1"
rhs: RSHI1(rc12,reg)        "%0  >> %1"

rhs: ADDU1(reg,rc12)        "%0  +  %1"
rhs: ADDU1(rc12,reg)        "%0  +  %1"
rhs: SUBU1(reg,rc12)        "%0  -  %1"
rhs: SUBU1(rc12,reg)        "%0  -  %1"
rhs: MULU1(reg,rc12)        "%0  *  %1"
rhs: MULU1(rc12,reg)        "%0  *  %1"
rhs: BORU1(reg,rc12)        "%0  |  %1"
rhs: BORU1(rc12,reg)        "%0  |  %1"
rhs: BANDU1(reg,rc12)       "%0  &  %1"
rhs: BANDU1(rc12,reg)       "%0  &  %1"
rhs: BXORU1(reg,rc12)       "%0  ^  %1"
rhs: BXORU1(rc12,reg)       "%0  ^  %1"
rhs: LSHU1(reg,rc12)        "%0 <<< %1"
rhs: LSHU1(rc12,reg)        "%0 <<< %1"
rhs: RSHU1(reg,rc12)        "%0 >>> %1"
rhs: RSHU1(rc12,reg)        "%0 >>> %1"

acon: ADDRGP1               "# addr"

rhs: acon                   "%0"
rhs: rc20                   "%0"
rhs: ADDRFP1                "M - %a"
rhs: ADDRLP1                "%a + O"

rhs: ADDP1(reg,acon)        "%0  +  %1"
rhs: ADDP1(acon,reg)        "%1  +  %0"
rhs: ADDP1(reg,rc20)        "%0  +  %1"
rhs: SUBP1(reg,reg)         "%0  -  %1"

reg: INDIRI1(rhs)           "%c <- [%0]\n" 1
reg: INDIRU1(rhs)           "%c <- [%0]\n" 1
reg: INDIRP1(rhs)           "%c <- [%0]\n" 1

rc12: reg                   "%0"
rc12: con12                 "%0"
rc20: rc12                  "%0"
rc20: con20                 "%0"

reg: rhs                    "%c <- %0\n"  1
reg: LOADI1(reg)            "# move\n"  move(a)
reg: LOADU1(reg)            "# move\n"  move(a)
reg: LOADP1(reg)            "# move\n"  move(a)

reg: BCOMI1(rc12)           "%c <- ~%0\n" 1
reg: BCOMU1(rc12)           "%c <- ~%0\n" 1
reg: NEGI1(rc12)            "%c <- -%0\n" 1

reg: DIVU1(reg,reg)         "# div/mod \n" 5
reg: MODU1(reg,reg)         "# div/mod \n" 5
reg: DIVI1(reg,reg)         "# div/mod \n" 5
reg: MODI1(reg,reg)         "# div/mod \n" 5
reg: CVPU1(reg)             "# convert\n"  move(a)
reg: CVUP1(reg)             "# convert\n"  move(a)
reg: CVII1(reg)             "# convert\n"  move(a)
reg: CVIU1(reg)             "# convert\n"  move(a)
reg: CVUI1(reg)             "# convert\n"  move(a)
reg: CVUU1(reg)             "# convert\n"  move(a)

stmt: ASGNB(reg,INDIRB(reg))"# copy struct through %0 from %1 \n"
stmt: ASGNI1(rhs,reg)       "%1 -> [%0]\n" 2
stmt: ASGNU1(rhs,reg)       "%1 -> [%0]\n" 2
stmt: ASGNP1(rhs,reg)       "%1 -> [%0]\n" 2
stmt: ASGNI1(reg,rhs)       "[%0] <- %1\n" 2
stmt: ASGNU1(reg,rhs)       "[%0] <- %1\n" 2
stmt: ASGNP1(reg,rhs)       "[%0] <- %1\n" 2

stmt: ARGI1(reg)            "# arg\n"
stmt: ARGU1(reg)            "# arg\n"
stmt: ARGP1(reg)            "# arg\n"

stmt: JUMPV(rhs)            "P <- %0\n" 1
stmt: LABELV                "%a:\n"

stmt: EQI1(reg,rc12)        "N <- %0 == %1; P <- (@%a - (. + 1)) &  N + P\n" 2
stmt: NEI1(reg,rc12)        "N <- %0 == %1; P <- (@%a - (. + 1)) &~ N + P\n" 2

stmt: GEI1(reg,rc12)        "N <- %0 >= %1; P <- (@%a - (. + 1)) &  N + P\n" 2
stmt: GTI1(reg,rc12)        "N <- %0 >  %1; P <- (@%a - (. + 1)) &  N + P\n" 2
stmt: LEI1(reg,rc12)        "N <- %0 <= %1; P <- (@%a - (. + 1)) &  N + P\n" 2
stmt: LTI1(reg,rc12)        "N <- %0 <  %1; P <- (@%a - (. + 1)) &  N + P\n" 2

stmt: EQU1(reg,rc12)        "N <- %0 == %1; P <- (@%a - (. + 1)) &  N + P\n" 2
stmt: NEU1(reg,rc12)        "N <- %0 == %1; P <- (@%a - (. + 1)) &~ N + P\n" 2

stmt: GEU1(reg,reg)         "N <- %0 >= %1; N <- %0 @ 31 + N; N <- %1 @ 31 + N; N <- N @ 0; P <- (@%a - (. + 1)) &  N + P\n" 5
stmt: GTU1(reg,reg)         "N <- %0 >  %1; N <- %0 @ 31 + N; N <- %1 @ 31 + N; N <- N @ 0; P <- (@%a - (. + 1)) &  N + P\n" 5
stmt: LEU1(reg,reg)         "N <- %0 <= %1; N <- %0 @ 31 + N; N <- %1 @ 31 + N; N <- N @ 0; P <- (@%a - (. + 1)) &  N + P\n" 5
stmt: LTU1(reg,reg)         "N <- %0 <  %1; N <- %0 @ 31 + N; N <- %1 @ 31 + N; N <- N @ 0; P <- (@%a - (. + 1)) &  N + P\n" 5

reg:  CALLI1(rhs)           "[O] <- P + 1; P <- %0 // call %0\n" 2
reg:  CALLU1(rhs)           "[O] <- P + 1; P <- %0 // call %0\n" 2
reg:  CALLP1(rhs)           "[O] <- P + 1; P <- %0 // call %0\n" 2
stmt: CALLV(rhs)            "[O] <- P + 1; P <- %0 // call %0\n" 2

stmt: CALLI1(rhs)           "[O] <- P + 1; P <- %0 // call %0\n" 2
stmt: CALLU1(rhs)           "[O] <- P + 1; P <- %0 // call %0\n" 2
stmt: CALLP1(rhs)           "[O] <- P + 1; P <- %0 // call %0\n" 2

stmt: RETI1(reg)            "# ret\n"
stmt: RETU1(reg)            "# ret\n"
stmt: RETP1(reg)            "# ret\n"
%%
static void progbeg(int argc, char *argv[]) {
    int i;

    parseflags(argc, argv);
    for (i = REG_A; i < REG_max; i++)
        intreg[i] = mkreg((char*)reg_names[i], i, 1, IREG);

    intregw = mkwildcard(intreg);

#define REG_bit(x) (1<<REG_##x)
#define REG_or_bit(x) | REG_bit(x)
    vmask[IREG] = tmask[IREG] =
        (0 REGS(REG_or_bit)) &
        // A is special 0
        // M is frame pointer
        // N is condition register and temporary
        // O is stack pointer
        // P is program counter
        ~(REG_bit(A) | REG_bit(M) | REG_bit(N) | REG_bit(O) | REG_bit(P));
}

static Symbol rmap(int opk) {
    return intregw;
}

static void segment(int n) {
    static const char segnames[][5] = {
        [CODE] = "code",
        [DATA] = "data",
        [BSS]  = "BSS",
        [LIT]  = "lit",
    };
    print("/* segment : %s */\n", segnames[n]);
}

static void progend(void) {
    /* make the last instruction labelable but zero-sized */
    print(".zero 0\n");
}

static void target(Node p) {
    assert(p);
    switch (generic(p->op)) {
        case DIV:
        case MOD:
        case CALL:
            setreg(p, intreg[REG_B]);
            break;
        case RET:
            rtarget(p, 0, intreg[REG_B]);
            break;
        case CNST:
            if (range(p, 0, 0) == 0) {
                setreg(p, intreg[REG_A]);
                p->x.registered = 1;
            }
            break;
        default:
            ; // XXX ASGN+B, ARG+B
    }
}

static void clobber(Node p) {}

static void emit2(Node p) {
#define preg(f) ((f)[getregnum(p->x.kids[0])]->x.name)
    switch (generic(p->op)) {
        case CVI:
        case CVU:
        case LOAD: {
            char *dst = intreg[getregnum(p)]->x.name;
            char *src = preg(intreg);
            assert(opsize(p->op) <= opsize(p->x.kids[0]->op));
            if (dst != src)
                print("%s <- %s\n", dst, src);
            break;
        }
        case DIV:
        case MOD: {
            const char *op = generic(p->op) == DIV ? "div" : "mod";
            const char type = optype(p->op) == I ? 'i' : 'u';
            char *dst = intreg[getregnum(p)]->x.name;
            char *src = preg(intreg);
            print("%s -> [O + 1] // arg 0\n", src);
            print("%s -> [O + 2] // arg 1\n", dst);
            print("[O] <- P + 1; P <- (@_%c%s - (. + 1)) + P\n", type, op);
            break;
        }
        case ARG:
            // leave space for return address
            print("%s -> [O + %d] // arg %d\n",
                  preg(intreg), p->syms[2]->u.c.v.i - 1, p->x.argno);
            break;
        case ADDRG: {
            const char *name = p->syms[0]->x.name;
            // TODO mend this ugly hack for prepending '@' to bare labels
            print((name[0] == '(' ? "%s" : "(@%s - (. + 1)) + P"), name);
            break;
        }
        case ASGN:
            if (optype(p->op) == B) {
                print("M -> [O]\n");
                int tmpregs[] = { REG_N, REG_M, REG_A/* unused */ };
                blkcopy(getregnum(p->kids[0]), 0, getregnum(p->kids[1]->kids[0]), 0, p->syms[0]->u.c.v.i, tmpregs);
                print("M <- [O]\n");
            }
            break;
    }
}

static void doarg(Node p) {
    static int argnum;
    if (argoffset == 0)
        argnum = 0;
    p->x.argno = argnum++;
    p->syms[2] = intconst(mkactual(1, -p->syms[0]->u.c.v.i));
}

static void blkfetch(int k, int off, int reg, int tmp) {
    if (k != 1) abort(); // TODO k != 1
    print("%s <- [%s + %d]\n", intreg[tmp]->name, intreg[reg]->name, off);
}

static void blkstore(int k, int off, int reg, int tmp) {
    if (k != 1) abort(); // TODO k != 1
    print("%s -> [%s + %d]\n", intreg[tmp]->name, intreg[reg]->name, off);
}

static void blkloop(int dreg, int doff, int sreg, int soff,
    int size, int tmps[]) {
    int lab = genlabel(1);
    const char *sname = intreg[sreg]->name,
               *dname = intreg[dreg]->name;
    const char *names[] = {
        intreg[tmps[0]]->name,
        intreg[tmps[1]]->name,
        intreg[tmps[2]]->name
    };
    print("%s <- %s + %d\n", names[1], dname, size);
    print("L%d:\n", lab);
    // TODO unroll by two
    blkfetch(1, soff + 0, sreg, tmps[0]);
    blkstore(1, doff + 0, dreg, tmps[0]);
    print("%s <- %s + 1\n", sname, sname);
    print("%s <- %s + 1\n", dname, dname);
    print("%s <- %s < %s\n", names[0], dname, names[1]);
    print("P <- (@L%d - (. + 1)) & %s + P\n", lab, names[0]);
}

static void local(Symbol p) {
    if (askregvar(p, (*IR->x.rmap)(ttob(p->type))) == 0) {
        assert(p->sclass == AUTO);
        offset += p->type->size;
        p->x.offset = offset;
        p->x.name = stringd(offset);
    }
}

static void function(Symbol f, Symbol caller[], Symbol callee[], int n) {
    int i;

    print("%s:\n", f->x.name);
    usedmask[0] = usedmask[1] = 0;
    freemask[0] = vmask[IREG];
    freemask[1] = 0;
    offset = 1;
    for (i = 0; callee[i]; i++) {
        Symbol p = callee[i];
        Symbol q = caller[i];
        assert(q);
        p->x.offset = q->x.offset = offset;
        p->x.name = q->x.name = stringf("%d", p->x.offset);
        p->sclass = q->sclass = AUTO;
        offset++;
    }
    assert(caller[i] == 0);
    offset = maxoffset = 0;
    gencode(caller, callee);
    framesize = maxoffset;

    print("M <- O   // frame create\n");
    print("O <- O - %d\n", framesize + 1); // account for return address
    emitcode();
    print("O <- M   // frame destroy\n");
    print("P <- [O] // return\n\n");
}

static void defsymbol(Symbol p) {
    if (p->scope >= LOCAL && p->sclass == STATIC)
        p->x.name = stringf("L%d", genlabel(1));
    else if (p->generated)
        p->x.name = stringf("L%s", p->name);
    else if (p->scope == GLOBAL || p->sclass == EXTERN)
        p->x.name = stringf("_%s", p->name);
    else if (p->scope == CONSTANTS
    && (isint(p->type) || isptr(p->type))
    && p->name[0] == '0' && p->name[1] == 'x')
        p->x.name = stringf("%s", p->name);
    else
        p->x.name = p->name;
}
static void address(Symbol q, Symbol p, long n) {
    if (p->scope == GLOBAL || p->sclass == STATIC || p->sclass == EXTERN) {
        q->x.name = stringf("(@%s - (. + (1 + %D)))", p->x.name, n);
    } else {
        assert(n <= INT_MAX && n >= INT_MIN);
        q->x.offset = p->x.offset + n;
        q->x.name = stringd(q->x.offset);
    }
}

static void defconst(int suffix, int size, Value v) {
    print(".word 0x%x\n", (unsigned)v.u);
}

static void defaddress(Symbol p) {
    print(".word @%s\n", p->x.name);
}

static void defstring(int n, char *str) {
    /* TODO use .utf32 prettily (with escapes for non-printable characters) */
    char *s;

    for (s = str; s < str + n; s++)
        print(".word %d\n", (*s)&0377);
}

static void export(Symbol p) {
    print(".global %s\n", p->x.name);
}

static void import(Symbol p) {
    print("/* imports %s */\n", p->x.name);
}

static void global(Symbol p) {
    print("%s:\n", p->x.name);
}

static void space(int n) {
    print(".zero %d\n", n);
}

Interface tenyrIR = {
    1, 1, 0,    /* char */
    1, 1, 0,    /* short */
    1, 1, 0,    /* int */
    1, 1, 0,    /* long */
    1, 1, 1,    /* long long */
    1, 1, 1,    /* float */
    1, 1, 1,    /* double */
    1, 1, 1,    /* long double */
    1, 1, 0,    /* T * */
    0, 1, 0,    /* struct */
    1,          /* little_endian */
    1,          /* mulops_calls TODO separate * from / and % */
    0,          /* wants_callb */
    0,          /* wants_argb */
    1,          /* left_to_right */
    0,          /* wants_dag */
    0,          /* unsigned_char */
    address,
    blockbeg,
    blockend,
    defaddress,
    defconst,
    defstring,
    defsymbol,
    emit,
    export,
    function,
    gen,
    global,
    import,
    local,
    progbeg,
    progend,
    segment,
    space,
    0, 0, 0, 0, 0, 0, 0,
    {1, rmap,
        blkfetch, blkstore, blkloop,
        _label,
        _rule,
        _nts,
        _kids,
        _string,
        _templates,
        _isinstruction,
        _ntname,
        emit2,
        doarg,
        target,
        clobber,
}
};
/* vi:set ts=4 sw=4 et:*/
