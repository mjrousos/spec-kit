---
name: rhinomocks-upgrading
description: Upgrading projects that use RhinoMocks when migrating from .NET Framework to .NET Core.
---

# Upgrading RhinoMocks Dependencies

When upgrading .NET Framework projects depending on RhinoMocks to .NET Core, replace RhinoMocks with Moq as RhinoMocks does not support .NET Core.

## Steps to Upgrade

1. **Plan per project**
	- Remove the RhinoMocks package/reference.
	- Add the latest `Moq` NuGet package.
	- Build to surface all RhinoMocks usages that must be ported.
2. **Work file-by-file**
	- For each compiler error, rewrite the RhinoMocks construct to its Moq counterpart per the mappings below.
	- Keep Arrange/Act/Assert structure intact; only change mocking syntax.
3. **Verify continuously**
	- Rebuild after each file or logical batch.
	- Keep/adjust existing tests so behavior stays identical.

## RhinoMocks → Moq Mappings
Use these substitutions whenever you encounter the RhinoMocks API.

| Intent | RhinoMocks Pattern | Moq Replacement |
| --- | --- | --- |
| Create mock | `MockRepository.GenerateMock<IMyType>()` | `var myTypeMock = new Mock<IMyType>(); var myType = myTypeMock.Object;` |
| Create stub/fake | `MockRepository.GenerateStub<IMyType>()` | Same as above; use `MockBehavior.Loose` if you need Rhino-style lenient behavior. |
| Expect method call with return | `mock.Expect(x => x.Do(Arg<T>.Is.Anything)).Return(value);` | `mock.Setup(x => x.Do(It.IsAny<T>())).Returns(value);` |
| Constrain argument | `Arg<T>.Is.Equal(value)` / `Arg<T>.Matches(predicate)` | `It.Is<T>(x => x == value)` / `It.Is<T>(predicate)` |
| Out parameter | `Arg<T>.Out(outValue).Dummy` | `mock.Setup(x => x.Do(out It.Ref<T>.IsAny)).Callback((out T v) => v = outValue);` |
| Ref parameter | `Arg<T>.Ref(Is.Equal(inValue), outValue).Dummy` | `mock.Setup(x => x.Do(ref It.Ref<T>.IsAny)).Callback((ref T v) => v = outValue);` |
| Throw exception | `.Throw(new InvalidOperationException())` | `.Throws<InvalidOperationException>()` or `.Throws(new InvalidOperationException())` |
| Verify call count | `.Repeat.Once()` / `.Repeat.Twice()` etc. | After `Act`, call `mock.Verify(x => x.Do(It.IsAny<T>()), Times.Once());` (reuse the same expression when possible). |
| Sequential returns | `Expect(...).Return(first).Repeat.Once(); Expect(...).Return(second);` | Use `var sequence = new MockSequence(); mock.InSequence(sequence).Setup(...).Returns(first); mock.InSequence(sequence).Setup(...).Returns(second);` |
| Events | `mock.Raise(x => x.SomeEvent += null, sender, args);` | `mock.Raise(x => x.SomeEvent += null, sender, args);` (same API) |

## Detailed Rewrite Checklist
1. **Search & Replace Imports**
	- Remove `using Rhino.Mocks;` and related aliases (`using Is = Rhino.Mocks.Constraints.Is;`).
	- Add `using Moq;` and `using Moq.Language.Flow;` (if needed for advanced scenarios).
	- These samples compile with C# 7.3 (net48 default). Avoid newer language constructs (e.g., `is not` patterns) and use helper delegates for `Callback` with `ref`/`out` parameters when Moq expects concrete delegate types.
2. **Mock Creation**
	- Replace `MockRepository` usage with direct `new Mock<T>()` instances.
	- Expose `.Object` wherever the concrete interface/class instance is required.
3. **Behavior Definition**
	- Convert every `.Expect`/`.Stub` chain to a `.Setup` (optionally `.SetupSequence`).
	- Translate `Arg<T>` helpers to the appropriate `It.*` helpers.
4. **Execution & Verification**
	- Remove `mockRepository.Replay()` / `VerifyAllExpectations()` calls.
	- Add explicit `mock.Verify(...)` calls for interactions that used `.Repeat` or `AssertWasCalled`.
5. **Partial/Concrete Types**
	- Prefer `new Mock<ConcreteType>() { CallBase = true }` to mimic Rhino partial mocks.
	- For constructors, pass args via `new Mock<ConcreteType>(ctorArgs)`.
	- When you need to raise virtual events on partial mocks, capture handlers via `SetupAdd`/`SetupRemove` before invoking them; `mock.Raise` alone will not fire events when `CallBase` is true.
6. **Sequential / Stateful Behavior**
	- Map Rhino `WhenCalled` delegates to Moq `.Callback(...)`.
	- Use `MockSequence` when the Rhino version relied on ordered expectations.
7. **Test Assertions**
	- Keep existing `Assert.*`/Should statements untouched; only adjust the mocked behavior feeding those assertions.

## Quality Gates for the Agent
- Do not change test intent; only alter mocking code.
- Run `dotnet test` (or equivalent) after the migration to ensure parity.
- If RhinoMocks offered functionality Moq lacks directly, document the limitation in code comments and choose the closest supported behavior.

These steps encapsulate the article’s key advice—remove RhinoMocks, introduce Moq, and methodically translate each mocking construct while preserving the AAA structure and verifying after every change.