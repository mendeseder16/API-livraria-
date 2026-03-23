# API-livraria-

Projeto completo em .NET (C#) para uma API REST que gerencie livros de uma livraria, com:

- CRUD completo para livros
- Validações (DTOs, FluentValidation)
- Documentação via Swagger / OpenAPI
- Uso de herança entre classes para organizar o domínio (ex.: entidade base, livros físicos e e-books)
- Camadas simples: Domain, DTOs, Services/Repositories (in-memory para exemplo) e Controllers
- Exemplos de entidades, mapeamento (AutoMapper), e tratamento de erros

Vou apresentar:
1) Estrutura do projeto (pastas/arquivos)
2) Código completo dos arquivos principais (Program.cs, modelos, DTOs, validators, repositório, serviços, controllers)
3) Configuração do Swagger e AutoMapper
4) Como rodar e testar
5) Melhorias possíveis e observações

Observação: o código usa .NET 7+ (pode ser .NET 6 se preferir; indique que altere o TargetFramework). Usarei pacotes: AutoMapper, FluentValidation.AspNetCore, Swashbuckle.AspNetCore.

1) Estrutura sugerida do projeto
- Livraria.Api/ (projeto principal)
  - Program.cs
  - Controllers/
    - LivrosController.cs
  - Domain/
    - Entities/
      - BaseEntity.cs
      - Livro.cs
      - LivroFisico.cs
      - Ebook.cs
  - DTOs/
    - LivroCreateDto.cs
    - LivroUpdateDto.cs
    - LivroReadDto.cs
  - Validators/
    - LivroCreateDtoValidator.cs
    - LivroUpdateDtoValidator.cs
  - Profiles/
    - LivroProfile.cs
  - Repositories/
    - ILivroRepository.cs
    - LivroRepository.cs
  - Services/
    - ILivroService.cs
    - LivroService.cs
  - Middlewares/
    - ExceptionMiddleware.cs
  - Properties/launchSettings.json (opcional)

2) Código dos arquivos principais

Program.cs
```csharp
using FluentValidation;
using FluentValidation.AspNetCore;
using Livraria.Api.Middlewares;
using Livraria.Api.Profiles;
using Livraria.Api.Repositories;
using Livraria.Api.Services;
using Microsoft.OpenApi.Models;

var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers()
    .AddFluentValidation(fv => fv.RegisterValidatorsFromAssemblyContaining<Program>());

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "Livraria API", Version = "v1" });
});

// DI: repository and service
builder.Services.AddSingleton<ILivroRepository, LivroRepository>();
builder.Services.AddScoped<ILivroService, LivroService>();

// AutoMapper
builder.Services.AddAutoMapper(typeof(LivroProfile).Assembly);

var app = builder.Build();

// Middleware para tratamento de exceções centralizado
app.UseMiddleware<ExceptionMiddleware>();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "Livraria API v1"));
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

Domain/Entities/BaseEntity.cs
```csharp
using System;

namespace Livraria.Api.Domain.Entities
{
    public abstract class BaseEntity
    {
        public Guid Id { get; set; } = Guid.NewGuid();
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
        public DateTime? UpdatedAt { get; set; }
    }
}
```

Domain/Entities/Livro.cs
```csharp
using System;

namespace Livraria.Api.Domain.Entities
{
    // Entidade base para livros (herança)
    public abstract class Livro : BaseEntity
    {
        public string Titulo { get; set; } = null!;
        public string Autor { get; set; } = null!;
        public DateTime DataPublicacao { get; set; }
        public decimal Preco { get; set; }
        public string ISBN { get; set; } = null!;
    }
}
```

Domain/Entities/LivroFisico.cs
```csharp
namespace Livraria.Api.Domain.Entities
{
    public class LivroFisico : Livro
    {
        public int NumeroPaginas { get; set; }
        public double PesoKg { get; set; }
    }
}
```

Domain/Entities/Ebook.cs
```csharp
namespace Livraria.Api.Domain.Entities
{
    public class Ebook : Livro
    {
        public string Formato { get; set; } = "PDF"; // PDF, EPUB, MOBI...
        public string UrlDownload { get; set; } = null!;
    }
}
```

DTOs/LivroCreateDto.cs
```csharp
using System;

namespace Livraria.Api.DTOs
{
    public class LivroCreateDto
    {
        public string Titulo { get; set; } = null!;
        public string Autor { get; set; } = null!;
        public DateTime DataPublicacao { get; set; }
        public decimal Preco { get; set; }
        public string ISBN { get; set; } = null!;
        // Tipo simples: "fisico" ou "ebook"
        public string Tipo { get; set; } = "fisico";
        // Campos opcionais para cada tipo
        public int? NumeroPaginas { get; set; }
        public double? PesoKg { get; set; }
        public string? Formato { get; set; }
        public string? UrlDownload { get; set; }
    }
}
```

DTOs/LivroUpdateDto.cs
```csharp
using System;

namespace Livraria.Api.DTOs
{
    public class LivroUpdateDto
    {
        public string? Titulo { get; set; }
        public string? Autor { get; set; }
        public DateTime? DataPublicacao { get; set; }
        public decimal? Preco { get; set; }
        public string? ISBN { get; set; }
        public int? NumeroPaginas { get; set; }
        public double? PesoKg { get; set; }
        public string? Formato { get; set; }
        public string? UrlDownload { get; set; }
    }
}
```

DTOs/LivroReadDto.cs
```csharp
using System;

namespace Livraria.Api.DTOs
{
    public class LivroReadDto
    {
        public Guid Id { get; set; }
        public string Titulo { get; set; } = null!;
        public string Autor { get; set; } = null!;
        public DateTime DataPublicacao { get; set; }
        public decimal Preco { get; set; }
        public string ISBN { get; set; } = null!;
        public string Tipo { get; set; } = null!;
        // Campos específicos
        public int? NumeroPaginas { get; set; }
        public double? PesoKg { get; set; }
        public string? Formato { get; set; }
        public string? UrlDownload { get; set; }
        public DateTime CreatedAt { get; set; }
        public DateTime? UpdatedAt { get; set; }
    }
}
```

Validators/LivroCreateDtoValidator.cs
```csharp
using FluentValidation;
using Livraria.Api.DTOs;
using System;

namespace Livraria.Api.Validators
{
    public class LivroCreateDtoValidator : AbstractValidator<LivroCreateDto>
    {
        public LivroCreateDtoValidator()
        {
            RuleFor(x => x.Titulo).NotEmpty().MaximumLength(200);
            RuleFor(x => x.Autor).NotEmpty().MaximumLength(100);
            RuleFor(x => x.DataPublicacao).LessThanOrEqualTo(DateTime.UtcNow);
            RuleFor(x => x.Preco).GreaterThanOrEqualTo(0);
            RuleFor(x => x.ISBN).NotEmpty().Length(10, 13);
            RuleFor(x => x.Tipo).NotEmpty().Must(t => t == "fisico" || t == "ebook")
                .WithMessage("Tipo deve ser 'fisico' ou 'ebook'.");

            When(x => x.Tipo == "fisico", () =>
            {
                RuleFor(x => x.NumeroPaginas).NotNull().GreaterThan(0);
                RuleFor(x => x.PesoKg).NotNull().GreaterThan(0);
            });

            When(x => x.Tipo == "ebook", () =>
            {
                RuleFor(x => x.Formato).NotEmpty();
                RuleFor(x => x.UrlDownload).NotEmpty().Must(u => Uri.IsWellFormedUriString(u, UriKind.Absolute))
                    .WithMessage("UrlDownload deve ser uma URL válida.");
            });
        }
    }
}
```

Validators/LivroUpdateDtoValidator.cs
```csharp
using FluentValidation;
using Livraria.Api.DTOs;
using System;

namespace Livraria.Api.Validators
{
    public class LivroUpdateDtoValidator : AbstractValidator<LivroUpdateDto>
    {
        public LivroUpdateDtoValidator()
        {
            When(x => x.Titulo != null, () => RuleFor(x => x.Titulo).NotEmpty().MaximumLength(200));
            When(x => x.Autor != null, () => RuleFor(x => x.Autor).NotEmpty().MaximumLength(100));
            When(x => x.DataPublicacao != null, () => RuleFor(x => x.DataPublicacao).LessThanOrEqualTo(DateTime.UtcNow));
            When(x => x.Preco != null, () => RuleFor(x => x.Preco).GreaterThanOrEqualTo(0));
            When(x => x.ISBN != null, () => RuleFor(x => x.ISBN).Length(10, 13));
            When(x => x.UrlDownload != null, () => RuleFor(x => x.UrlDownload).Must(u => Uri.IsWellFormedUriString(u, UriKind.Absolute)));
        }
    }
}
```

Profiles/LivroProfile.cs (AutoMapper)
```csharp
using AutoMapper;
using Livraria.Api.Domain.Entities;
using Livraria.Api.DTOs;
using System;

namespace Livraria.Api.Profiles
{
    public class LivroProfile : Profile
    {
        public LivroProfile()
        {
            CreateMap<LivroCreateDto, Livro>()
                .ConvertUsing((src, dest, ctx) =>
                {
                    if (src.Tipo == "ebook")
                    {
                        return new Ebook
                        {
                            Titulo = src.Titulo,
                            Autor = src.Autor,
                            DataPublicacao = src.DataPublicacao,
                            Preco = src.Preco,
                            ISBN = src.ISBN,
                            Formato = src.Formato ?? "PDF",
                            UrlDownload = src.UrlDownload ?? string.Empty
                        };
                    }
                    else
                    {
                        return new LivroFisico
                        {
                            Titulo = src.Titulo,
                            Autor = src.Autor,
                            DataPublicacao = src.DataPublicacao,
                            Preco = src.Preco,
                            ISBN = src.ISBN,
                            NumeroPaginas = src.NumeroPaginas ?? 0,
                            PesoKg = src.PesoKg ?? 0
                        };
                    }
                });

            CreateMap<Livro, LivroReadDto>()
                .ForMember(d => d.Tipo, opt => opt.MapFrom(src => src is Ebook ? "ebook" : "fisico"));

            CreateMap<LivroUpdateDto, Livro>()
                .ConvertUsing((src, dest, ctx) =>
                {
                    // Esse map não cria instâncias novas; será usado para aplicar atualizações em instância existente
                    return dest; // mapping manual feito no service
                });
        }
    }
}
```

Repositories/ILivroRepository.cs
```csharp
using Livraria.Api.Domain.Entities;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace Livraria.Api.Repositories
{
    public interface ILivroRepository
    {
        Task<IEnumerable<Livro>> GetAllAsync();
        Task<Livro?> GetByIdAsync(Guid id);
        Task AddAsync(Livro livro);
        Task UpdateAsync(Livro livro);
        Task DeleteAsync(Guid id);
    }
}
```

Repositories/LivroRepository.cs (in-memory)
```csharp
using Livraria.Api.Domain.Entities;
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace Livraria.Api.Repositories
{
    public class LivroRepository : ILivroRepository
    {
        private readonly ConcurrentDictionary<Guid, Livro> _store = new();

        public Task AddAsync(Livro livro)
        {
            _store[livro.Id] = livro;
            return Task.CompletedTask;
        }

        public Task DeleteAsync(Guid id)
        {
            _store.TryRemove(id, out _);
            return Task.CompletedTask;
        }

        public Task<IEnumerable<Livro>> GetAllAsync()
        {
            return Task.FromResult(_store.Values.AsEnumerable());
        }

        public Task<Livro?> GetByIdAsync(Guid id)
        {
            _store.TryGetValue(id, out var livro);
            return Task.FromResult(livro);
        }

        public Task UpdateAsync(Livro livro)
        {
            livro.UpdatedAt = DateTime.UtcNow;
            _store[livro.Id] = livro;
            return Task.CompletedTask;
        }
    }
}
```

Services/ILivroService.cs
```csharp
using Livraria.Api.DTOs;
using Livraria.Api.Domain.Entities;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace Livraria.Api.Services
{
    public interface ILivroService
    {
        Task<IEnumerable<LivroReadDto>> GetAllAsync();
        Task<LivroReadDto?> GetByIdAsync(Guid id);
        Task<LivroReadDto> CreateAsync(LivroCreateDto dto);
        Task<bool> UpdateAsync(Guid id, LivroUpdateDto dto);
        Task<bool> DeleteAsync(Guid id);
    }
}
```

Services/LivroService.cs
```csharp
using AutoMapper;
using Livraria.Api.DTOs;
using Livraria.Api.Domain.Entities;
using Livraria.Api.Repositories;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace Livraria.Api.Services
{
    public class LivroService : ILivroService
    {
        private readonly ILivroRepository _repo;
        private readonly IMapper _mapper;

        public LivroService(ILivroRepository repo, IMapper mapper)
        {
            _repo = repo;
            _mapper = mapper;
        }

        public async Task<LivroReadDto> CreateAsync(LivroCreateDto dto)
        {
            var entidade = _mapper.Map<Livro>(dto);
            await _repo.AddAsync(entidade);
            return _mapper.Map<LivroReadDto>(entidade);
        }

        public async Task<bool> DeleteAsync(Guid id)
        {
            var existing = await _repo.GetByIdAsync(id);
            if (existing == null) return false;
            await _repo.DeleteAsync(id);
            return true;
        }

        public async Task<IEnumerable<LivroReadDto>> GetAllAsync()
        {
            var list = await _repo.GetAllAsync();
            return list.Select(l => _mapper.Map<LivroReadDto>(l));
        }

        public async Task<LivroReadDto?> GetByIdAsync(Guid id)
        {
            var l = await _repo.GetByIdAsync(id);
            return l == null ? null : _mapper.Map<LivroReadDto>(l);
        }

        public async Task<bool> UpdateAsync(Guid id, LivroUpdateDto dto)
        {
            var existing = await _repo.GetByIdAsync(id);
            if (existing == null) return false;

            // Aplicar mudanças manualmente com atenção à herança
            if (dto.Titulo != null) existing.Titulo = dto.Titulo;
            if (dto.Autor != null) existing.Autor = dto.Autor;
            if (dto.DataPublicacao.HasValue) existing.DataPublicacao = dto.DataPublicacao.Value;
            if (dto.Preco.HasValue) existing.Preco = dto.Preco.Value;
            if (dto.ISBN != null) existing.ISBN = dto.ISBN;

            if (existing is LivroFisico lf)
            {
                if (dto.NumeroPaginas.HasValue) lf.NumeroPaginas = dto.NumeroPaginas.Value;
                if (dto.PesoKg.HasValue) lf.PesoKg = dto.PesoKg.Value;
            }
            else if (existing is Ebook eb)
            {
                if (dto.Formato != null) eb.Formato = dto.Formato;
                if (dto.UrlDownload != null) eb.UrlDownload = dto.UrlDownload;
            }

            existing.UpdatedAt = DateTime.UtcNow;
            await _repo.UpdateAsync(existing);
            return true;
        }
    }
}
```

Middlewares/ExceptionMiddleware.cs
```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using System;
using System.Net;
using System.Text.Json;
using System.Threading.Tasks;

namespace Livraria.Api.Middlewares
{
    public class ExceptionMiddleware
    {
        private readonly RequestDelegate _next;
        private readonly ILogger<ExceptionMiddleware> _logger;

        public ExceptionMiddleware(RequestDelegate next, ILogger<ExceptionMiddleware> logger)
        {
            _next = next;
            _logger = logger;
        }

        public async Task InvokeAsync(HttpContext httpContext)
        {
            try
            {
                await _next(httpContext);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Unhandled exception");
                httpContext.Response.ContentType = "application/json";
                httpContext.Response.StatusCode = (int)HttpStatusCode.InternalServerError;
                var result = JsonSerializer.Serialize(new { error = "Ocorreu um erro interno." });
                await httpContext.Response.WriteAsync(result);
            }
        }
    }
}
```

Controllers/LivrosController.cs
```csharp
using Livraria.Api.DTOs;
using Livraria.Api.Services;
using Microsoft.AspNetCore.Mvc;
using System;
using System.Threading.Tasks;

namespace Livraria.Api.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class LivrosController : ControllerBase
    {
        private readonly ILivroService _service;

        public LivrosController(ILivroService service)
        {
            _service = service;
        }

        [HttpGet]
        public async Task<IActionResult> GetAll()
        {
            var itens = await _service.GetAllAsync();
            return Ok(itens);
        }

        [HttpGet("{id:guid}")]
        public async Task<IActionResult> GetById(Guid id)
        {
            var item = await _service.GetByIdAsync(id);
            if (item == null) return NotFound();
            return Ok(item);
        }

        [HttpPost]
        public async Task<IActionResult> Create([FromBody] LivroCreateDto dto)
        {
            var created = await _service.CreateAsync(dto);
            return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
        }

        [HttpPut("{id:guid}")]
        public async Task<IActionResult> Update(Guid id, [FromBody] LivroUpdateDto dto)
        {
            var ok = await _service.UpdateAsync(id, dto);
            if (!ok) return NotFound();
            return NoContent();
        }

        [HttpDelete("{id:guid}")]
        public async Task<IActionResult> Delete(Guid id)
        {
            var ok = await _service.DeleteAsync(id);
            if (!ok) return NotFound();
            return NoContent();
        }
    }
}
```

3) Pacotes NuGet a instalar
- AutoMapper.Extensions.Microsoft.DependencyInjection
- FluentValidation.AspNetCore
- Swashbuckle.AspNetCore

Comandos (CLI):
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
dotnet add package FluentValidation.AspNetCore
dotnet add package Swashbuckle.AspNetCore

4) Como rodar e testar
- dotnet restore
- dotnet run
- Abrir https://localhost:{port}/swagger para ver a documentação e testar os endpoints.
- Exemplos de payloads:

Criar livro físico (POST /api/livros)
{
  "titulo": "C# em Ação",
  "autor": "Autor Exemplo",
  "dataPublicacao": "2020-05-10T00:00:00Z",
  "preco": 99.90,
  "isbn": "1234567890123",
  "tipo": "fisico",
  "numeroPaginas": 450,
  "pesoKg": 0.8
}

Criar ebook
{
  "titulo": "Ebook Exemplo",
  "autor": "Autor Exemplo",
  "dataPublicacao": "2021-01-01T00:00:00Z",
  "preco": 39.90,
  "isbn": "1234567890",
  "tipo": "ebook",
  "formato": "EPUB",
  "urlDownload": "https://exemplo.com/download/ebook"
}

5) Melhorias e considerações
- Persistência real: trocar repositório in-memory por EF Core com migrations e SQL Server / PostgreSQL.
- Autenticação/Autorização: adicionar JWT, políticas por role.
- Versionamento de API via atributos e Swagger.
- Paginação, filtros e ordenação para GET /livros.
- Testes unitários e de integração (xUnit + WebApplicationFactory).
- Mapeamento mais robusto para updates (Patch/JSON Patch) e DTOs separados por tipo.
- Políticas para ISBN (validações específicas).
- Tratamento de concurrency (rowversion) quando usar BD relacional.
