using System.Collections.Immutable;
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp;
using Microsoft.CodeAnalysis.CSharp.Syntax;
using Microsoft.CodeAnalysis.Diagnostics;

namespace BugHunter.Analyzers.CmsApiReplacementRules.AnalyzersX
{
    [DiagnosticAnalyzer(LanguageNames.CSharp)]
    public class HttpResponseCookiesAnalyzer : DiagnosticAnalyzer
    {
        private static readonly DiagnosticDescriptor Rule = new DiagnosticDescriptor("BH1003",
            title: "'Response.Cookies' should not be used.",
            messageFormat: "'{0}' should not be used. Use 'CookiesHelper' instead.",
            category: "CmsApiReplacements",
            defaultSeverity: DiagnosticSeverity.Warning,
            isEnabledByDefault: true,
            description: "Do not access Cookies property of HttpResponse directly. Use CookiesHelper instead.");

        public override ImmutableArray<DiagnosticDescriptor> SupportedDiagnostics
            => ImmutableArray.Create(Rule, Rule);

        public override void Initialize(AnalysisContext context)
        {
            context.RegisterSyntaxNodeAction(Analyze, SyntaxKind.SimpleMemberAccessExpression);
        }

        private void Analyze(SyntaxNodeAnalysisContext syntaxNodeAnalysisContext)
        {
            var memberAccess = syntaxNodeAnalysisContext.Node as MemberAccessExpressionSyntax;
            if (memberAccess == null || memberAccess.Name.Identifier.ValueText != "Cookies") 
            {
                return;
            }

            var semanticModel = syntaxNodeAnalysisContext.SemanticModel;
            var accessedType = semanticModel.GetTypeInfo(memberAccess.Expression).Type as INamedTypeSymbol;
            if (accessedType == null || accessedType.ToString() != "System.Web.HttpResponse")
            {
                return;
            }

            var diagnostic = Diagnostic.Create(Rule, memberAccess.GetLocation(), memberAccess);
            syntaxNodeAnalysisContext.ReportDiagnostic(diagnostic);
        }
    }
}
