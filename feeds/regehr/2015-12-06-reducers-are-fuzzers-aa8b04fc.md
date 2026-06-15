---
title: Reducers are Fuzzers
url: https://blog.regehr.org/archives/1284
published: "2015-12-06T22:55:24Z"
feed: regehr
guid: http://blog.regehr.org/?p=1284
---

# Reducers are Fuzzers

A test case isn’t just a test case: it lives in the (generally extremely large) space of inputs for the software system you are testing. If we have a test case that triggers a bug, here’s one way we can look at it:

![](https://blog.regehr.org/extra_files/reducers-are-fuzzers/web1.png)

The set of test cases triggering a bug is a useful notion since we can search it. For example, a test case reducer is a program that searches for the smallest test case triggering a bug. It requires a way to transform a test case into a smaller one, for example by deleting part of it. The new variant of the test case may or may not trigger the bug. The process goes like this:

![](https://blog.regehr.org/extra_files/reducers-are-fuzzers/web2.png)

I’ve spent a lot of time watching reducers run, and one thing I’ve noticed is that the reduction process often triggers bugs unrelated to the bug that is the subject of the reduction:

![](https://blog.regehr.org/extra_files/reducers-are-fuzzers/web3.png)

Sometimes this is undesirable, such as when a lax interestingness test permits the reduction of one bug to get hijacked by a different bug. This happens all the time when reducing segfaults, which are hard to tell apart. But on the other hand, if we’re looking for bugs then this phenomenon is a useful one.

It seems a bit counterintuitive that test case reduction would lead to the discovery of new bugs since we might expect that the space of inputs to a well-tested software system is mostly non-bug-triggering with a few isolated pockets of bug-triggering inputs scattered here and there. I am afraid that that view might not be realistic. Rather, all of the inputs we usually see occupy a tiny portion of the space of inputs, and it is surrounded by huge overlapping clouds of bug-triggering inputs. Fuzzers can push the boundaries of the space of inputs that we can test, but not by as much as people generally think. Proofs remain the only way to actually show that a piece of software does the right thing any significant chunk of its input space. But I digress. The important fact is that reducers are decently effective mutation-based fuzzers.

In the rest of this post I’d like to push that idea a bit farther by doing a reduction that doesn’t correspond to a bug and seeing if we can find some bugs along the way. We’ll start with this exciting C++ program

```
#include <iostream>

int main() {
  std::cout << "Hello World!++" << std::endl;
}

```

and we’ll reduce it under the criterion that it remains a viable Hello World implementation. First we’ll preprocess and then use [delta’s](http://delta.tigris.org/) topformflat to smoosh everything together nicely:

**```** **g++ -std=c++11 -E -P -O3 hello-orig.cpp | topformflat > hello.cpp**

**```**

You might be saying to yourself something like “it sure is stupid that John is passing an optimization flag to the preprocessor,” but trust me that it actually does change the emitted code. I didn’t check if it makes a difference here but I’ve learned to avoid trouble by just passing the same flags to the preprocessor as to the compiler.

Anyhow, the result is 550 KB of goodness:

![](https://blog.regehr.org/extra_files/reducers-are-fuzzers/plusplus.png)

Here’s the code that checks if a variant is interesting — that is, if it acts like a Hello World implementation:

**```** **g++ -O3 -std=c++11 -w hello.cpp >/dev/null 2>&1 &&** **ulimit -t 1 &&** **./a.out | grep Hello**

**```**

The ulimit is necessary because infinite loops sometimes get into the program that is being reduced.

To find compiler crashes we’ll need a bit more elaborate of a test:

**```** **if** **g++ -O3 -std=c++11 -w hello.cpp >compiler.out 2>&1** **then** **ulimit -t 1 &&** **./a.out | grep Hello** **else** **if** **grep 'internal compiler error' compiler.out** **then** **exit 101** **else** **exit 1** **fi** **fi**

**```**

When the compiler fails we look at its output and, if it contains evidence of a compiler bug, exit with code 101, which will tell C-Reduce that it should save a copy of the input files that made this happen.

The compiler we’ll use is g++ r231221, the development head from December 3 2015. Let’s get things going:

**```** **creduce --nokill --also-interesting 101 --no-default-passes \** **--add-pass pass_clex rm-tok-pattern-4 10 ../test.sh hello.cpp**

**```**

The `-also-interesting 101` option indicates that the interestingness test will use process exit code 101 to tell C-Reduce to make a snapshot of the directory containing the files being reduced, so we can look at it later. `--no-default-passes` clears C-Reduce’s pass schedule and `-add-pass pass_clex rm-tok-pattern-4 10` add a single pass that uses a small sliding window to remove tokens from the test case. The issue here is that not all of C-Reduce’s passes are equally effective at finding bugs. Some passes, such as the one that removes dead variables and the one that removes dead functions, will probably never trigger a compiler bug. Other passes, such as the one that removes chunks of lines from a test case, eliminate text from the test case so rapidly that effective fuzzing doesn’t happen. There are various ways to deal with this problem, such as probabilistically rejecting improvements or rejecting improvements that are too large, but for this post I’ve chosen the simple expedient of running just one pass that (1) makes progress very slowly and (2) seems to be a good fuzzer.

The dynamics of this sort of run are interesting: as the test case walks around the space of programs, you can actually see it brush up against a compiler bug and then wander off into the weeds again:

![](https://blog.regehr.org/extra_files/reducers-are-fuzzers/progress.png)

The results are decent: after about 24 hours, C-Reduce caused many segfaults in g++ and triggered six different internal compiler errors: [1](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=68473), [2](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=68722), [3](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=68723), [4](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=68724), [5](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=68726), [6](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=68728). One of these was already reported, another looks probably like a duplicate, and four appear to be new.

I did a similar run against Clang++ r254595, again the development head from December 3. This produced segfaults and also triggered 25 different assertions:

**```** **llvm/include/llvm/Support/Casting.h:230: typename cast_retty::ret_type llvm::cast(Y &) [X = clang::FunctionProtoType, Y = clang::QualType]: Assertion `isa(Val) && "cast() argument of incompatible type!"' failed.** **llvm/include/llvm/Support/Casting.h:95: static bool llvm::isa_impl_cl::doit(const From *) [To = clang::FunctionTemplateDecl, From = const clang::Decl *]: Assertion `Val && "isa<> used on a null pointer"' failed.** **llvm/tools/clang/lib/AST/Decl.cpp:2134: clang::APValue *clang::VarDecl::evaluateValue(SmallVectorImpl &) const: Assertion `!Init->isValueDependent()' failed.** **llvm/tools/clang/lib/AST/Decl.cpp:2181: bool clang::VarDecl::checkInitIsICE() const: Assertion `!Init->isValueDependent()' failed.** **llvm/tools/clang/lib/AST/ExprCXX.cpp:451: static clang::DependentScopeDeclRefExpr *clang::DependentScopeDeclRefExpr::Create(const clang::ASTContext &, clang::NestedNameSpecifierLoc, clang::SourceLocation, const clang::DeclarationNameInfo &, const clang::TemplateArgumentListInfo *): Assertion `QualifierLoc && "should be created for dependent qualifiers"' failed.** **llvm/tools/clang/lib/AST/../../include/clang/AST/TypeNodes.def:98: clang::TypeInfo clang::ASTContext::getTypeInfoImpl(const clang::Type *) const: Assertion `!T->isDependentType() && "should not see dependent types here"' failed.** **llvm/tools/clang/lib/CodeGen/CodeGenModule.cpp:623: llvm::StringRef clang::CodeGen::CodeGenModule::getMangledName(clang::GlobalDecl): Assertion `II && "Attempt to mangle unnamed decl."' failed.** **llvm/tools/clang/lib/CodeGen/../../include/clang/AST/Expr.h:134: void clang::Expr::setType(clang::QualType): Assertion `(t.isNull() || !t->isReferenceType()) && "Expressions can't have reference type"' failed.** **llvm/tools/clang/lib/Lex/PPCaching.cpp:101: void clang::Preprocessor::AnnotatePreviousCachedTokens(const clang::Token &): Assertion `CachedTokens[CachedLexPos-1].getLastLoc() == Tok.getAnnotationEndLoc() && "The annotation should be until the most recent cached token"' failed.** **llvm/tools/clang/lib/Parse/../../include/clang/Parse/Parser.h:2256: void clang::Parser::DeclaratorScopeObj::EnterDeclaratorScope(): Assertion `!EnteredScope && "Already entered the scope!"' failed.** **llvm/tools/clang/lib/Sema/../../include/clang/AST/DeclTemplate.h:1707: void clang::ClassTemplateSpecializationDecl::setInstantiationOf(clang::ClassTemplatePartialSpecializationDecl *, const clang::TemplateArgumentList *): Assertion `!SpecializedTemplate.is() && "Already set to a class template partial specialization!"' failed.** **llvm/tools/clang/lib/Sema/../../include/clang/Sema/Lookup.h:460: clang::NamedDecl *clang::LookupResult::getFoundDecl() const: Assertion `getResultKind() == Found && "getFoundDecl called on non-unique result"' failed.** **llvm/tools/clang/lib/Sema/SemaDecl.cpp:10455: clang::Decl *clang::Sema::ActOnParamDeclarator(clang::Scope *, clang::Declarator &): Assertion `S->isFunctionPrototypeScope()' failed.** **llvm/tools/clang/lib/Sema/SemaDeclCXX.cpp:11373: ExprResult clang::Sema::BuildCXXDefaultInitExpr(clang::SourceLocation, clang::FieldDecl *): Assertion `Lookup.size() == 1' failed.** **llvm/tools/clang/lib/Sema/SemaExpr.cpp:2274: ExprResult clang::Sema::ActOnIdExpression(clang::Scope *, clang::CXXScopeSpec &, clang::SourceLocation, clang::UnqualifiedId &, bool, bool, std::unique_ptr, bool, clang::Token *): Assertion `R.getAsSingle() && "There should only be one declaration found."' failed.** **llvm/tools/clang/lib/Sema/SemaExprCXX.cpp:2272: clang::FunctionDecl *clang::Sema::FindUsualDeallocationFunction(clang::SourceLocation, bool, clang::DeclarationName): Assertion `Matches.size() == 1 && "unexpectedly have multiple usual deallocation functions"' failed.** **llvm/tools/clang/lib/Sema/SemaExprCXX.cpp:6663: ExprResult clang::Sema::CorrectDelayedTyposInExpr(clang::Expr *, clang::VarDecl *, llvm::function_ref): Assertion `TyposInContext < ~0U && "Recursive call of CorrectDelayedTyposInExpr"' failed.** **llvm/tools/clang/lib/Sema/SemaExprMember.cpp:91: IMAKind ClassifyImplicitMemberAccess(clang::Sema &, const clang::LookupResult &): Assertion `!R.empty() && (*R.begin())->isCXXClassMember()' failed.** **llvm/tools/clang/lib/Sema/SemaLookup.cpp:1904: bool clang::Sema::LookupQualifiedName(clang::LookupResult &, clang::DeclContext *, bool): Assertion `(!isa(LookupCtx) || LookupCtx->isDependentContext() || cast(LookupCtx)->isCompleteDefinition() || cast(LookupCtx)->isBeingDefined()) && "Declaration context must already be complete!"' failed.** **llvm/tools/clang/lib/Sema/SemaLookup.cpp:2729: Sema::SpecialMemberOverloadResult *clang::Sema::LookupSpecialMember(clang::CXXRecordDecl *, clang::Sema::CXXSpecialMember, bool, bool, bool, bool, bool): Assertion `CanDeclareSpecialMemberFunction(RD) && "doing special member lookup into record that isn't fully complete"' failed.** **llvm/tools/clang/lib/Sema/SemaOverload.cpp:11671: ExprResult clang::Sema::CreateOverloadedBinOp(clang::SourceLocation, unsigned int, const clang::UnresolvedSetImpl &, clang::Expr *, clang::Expr *): Assertion `Result.isInvalid() && "C++ binary operator overloading is missing candidates!"' failed.** **llvm/tools/clang/lib/Sema/SemaTemplate.cpp:2906: ExprResult clang::Sema::BuildTemplateIdExpr(const clang::CXXScopeSpec &, clang::SourceLocation, clang::LookupResult &, bool, const clang::TemplateArgumentListInfo *): Assertion `!R.empty() && "empty lookup results when building templateid"' failed.** **llvm/tools/clang/lib/Sema/SemaTemplateDeduction.cpp:609: (anonymous namespace)::PackDeductionScope::PackDeductionScope(clang::Sema &, clang::TemplateParameterList *, SmallVectorImpl &, clang::sema::TemplateDeductionInfo &, clang::TemplateArgument): Assertion `!Packs.empty() && "Pack expansion without unexpanded packs?"' failed.** **llvm/tools/clang/lib/Sema/SemaTemplateInstantiate.cpp:2781: llvm::PointerUnion *clang::LocalInstantiationScope::findInstantiationOf(const clang::Decl *): Assertion `isa(D) && "declaration not instantiated in this scope"' failed.** **llvm/tools/clang/lib/Sema/SemaTemplateVariadic.cpp:290: bool clang::Sema::DiagnoseUnexpandedParameterPack(clang::Expr *, clang::Sema::UnexpandedParameterPackContext): Assertion `!Unexpanded.empty() && "Unable to find unexpanded parameter packs"' failed.**

**```**

I have to admit that I felt a bit overwhelmed by 25 potential bug reports, and I haven’t reported any of these yet. My guess is that a number of them are already in the bugzilla since people have been fuzzing Clang lately. Anyway, I’ll try to get around to reducing and reporting these. Really, this all needs to be automated so that when subsequent reductions find still more bugs, these just get added to the queue of reductions to run.

If you were interested in reproducing these results, or in trying something similar, you would want to use [C-Reduce’s master branch](https://github.com/csmith-project/creduce). I ran everything on an Ubuntu 14.04 box. While preparing this post I found that different C-Reduce command line options produced widely varying numbers and kinds of crashes.

Regarding previous work, I believe — but couldn’t find written down — that the [CERT BFF](https://www.cert.org/vulnerability-analysis/tools/bff.cfm?) watches out for new crashes when running its reducer. In a [couple](http://www.cs.utah.edu/~regehr/papers/icst14.pdf) of [papers](http://www.cs.utah.edu/~regehr/papers/mintest.pdf) written by people like Alex Groce and me, we discussed the fact that reducers often slip from triggering one bug to another.

The new thing in this post is to show that triggering new bugs while reducing isn’t just some side effect. Rather, we can go looking for trouble, and we can do it without being given a bug to reduce in the first place. A key enabler for easy bug-finding with C-Reduce was finding a simple communication mechanism by which the interestingness test can give C-Reduce a bit of out-of-band information that a variant should be saved for subsequent inspection. I’m not trying to claim that reducers are awesome fuzzers, but on the other hand, it might be a little bit awesome that mutating Hello World resulted in triggering 25 different assertion violations in a mature and high-quality compiler. I bet we’d have done even better by starting with a nice big fat Boost application.
